---
title: "engine abstraction and caching"
date: 2023-06-01
mathjax: true
draft: false
series:
tags: [library]
---

KRPC already provides some API to interface with part and modules. SpaceLib create a wrapper around it, so that it is easier to use and test. In this post, I will cover how I implemented `Engine` module in SpaceLib, providing a high level abstraction over KRPC.jl's `SCR.Engine` related calls.

## The KRPC API

Let's begin with what you should do to obtain part and modules of an engine and use it.

```julia
using KRPC
import KRPC.Interface.SpaceCenter.RemoteTypes as SCR
import KRPC.Interface.SpaceCenter.Helpers as SCH

# things we should already have
conn = kerbal_connect(...)
space_center = SCR.SpaceCenter(conn)
ves = SCH.ActiveVessel(space_center)

# getting the part of interest
parts = SCH.Parts(ves)
part = SCH.WithTitle(parts, "MyPartName")[1]
engine = SCH.Engine(part)

# using it
SCH.Active!(engine, true)
@show SCH.Throttle(engine)
```

and then, to do something more complicated with it, it may be necessary to use actions/events or fields defined within its part module.

```julia
function getmodule(part::SCR.Part, name::String)::SCR.Module
    modules = SCH.Modules(part)
    for m ∈ modules
        if SCH.Name(m) == name
            return m
        end
    end
    error("Module $name not found")
end

rf_module = getmodule(part, "ModuleEnginesRF")
@show remaining_ignitions = SCH.GetFieldById(rf_module, "ignitions")
```

In my experience, any operation involving string takes much longer time in KRPC. In this example, I want know how many ignitions are left in the engine part. To do this, I need to loop through all known modules, and then query a field to get an answer. This entire process takes more than 1 second to finish.

If I save this part module and then re-use it later, then I can get significant speed boost. So why not do it across other KRPC objects?

## Caching Engine

Let's create a struct, `RealEngine` that will save the necessary modules and save them for later use.

```julia
# partmodule/engine.jl

# struct definition
struct RealEngine <: RealSingleEngine
    name::String
    part::SCR.Part
    engine::SCR.Engine
    module_realfuel::SCR.Module
    module_testflight::SCR.Module
    spooltime::Float32
    propellant::EnginePropellant
end

# constructor
function RealEngine(part::SCR.Part)
    name = SCH.Title(part)
    @debug "Indexing $name" _group=:index
    engine = SCH.Engine(part)
    rfm = getmodule(part, "ModuleEnginesRF")
    tfm = getmodule(part, "TestFlightReliability_EngineCycle")
    spool = spooltime(rfm)
    ṁ = static_mass_flow_rate(engine)
    ṁ ≤ 0 && error("Invalid massflow for $name")
    prop = EnginePropellant(engine, residual(rfm))
    return RealEngine(name, part, engine, rfm, tfm, spool, prop)
end

function ignitions(e::SingleEngine)
    value = SCH.GetFieldById(e.module_realfuel, "ignitions")
    return parse(Int64,value)
end
```

Wait, but there is more! I also need cache propellant information, not just the engine. I will need to know about how much burn time I have left for some maneuvers, so it's important to have make operation as fast as possible.

Burn time can be calculated by dividing current fuel mass (mₚ) with how much fuel I will be burning per second (ṁ).

$t = {m_p \over{\dot{m}}}$

I can calculate the mass flow rate [using ISP and thrust](https://www.grc.nasa.gov/www/k-12/airplane/specimp.html).

$I_{sp} = {F \over{\dot{m} \ g_0}}$

```julia
function static_mass_flow_rate(engine::SCR.Engine)
    thv = SCH.MaxVacuumThrust(engine)
    isp = SCH.VacuumSpecificImpulse(engine)
    return ṁ = thv / isp / g
end
```

In KSP the maximum mass flow rate will not change over time, so I only need to query it once and save it for later.

To get the mass of the propellant, I can loop through relevant resource containers and count the amount of fuel left in there.

```julia
engine = SCH.Engine(part)
props = SCH.Propellants(engine)
fuel_mass = 0
for p ∈ props
    fuel_mass += SCH.TotalResourceAvailable(p)
end
@show fuel_mass
```

This approach simply gets related resources and sum all remaining fuel. This does not work well when the fuel tank does not have perfect ratio of fuel and oxidizer. In Realism Overhaul, this happens often due to probabilistic consumption of engine resources. So, I need to take the consumption ratio into account. `SCH.Ratio` can be used to pull this ratio data from KRPC. It denotes how much "amount" will be used per unit time.

```julia
# engine will shut down when it runs out of at least 1 of its resources.
# find out which fuel has the worst burn time left.
times = Vector{Float64}()
masses = Vector{Float64}()
for p ∈ props
    amount = SCH.TotalResourceAvailable(p)
    push!(times, amount/SCH.Ratio(p))
    push!(masses, amount*SCH.Density(conn, SCH.Name(p)))
end
tmin = minimum(times)

# now we have the worst burn time. Let's re-construct effective fuel mass.
sum = 0
for (i, m) ∈ enumerate(masses)
    sum += tmin / times[i] * m
end
@show sum
```

Again, in this process I had to make a lot of queries to KRPC! To minimize number of queries, I can save static values and only call for the number I really need on the fly. To achieve this, I created two more structs, `EnginePropellant` and `StoredPropellant`.

## Caching Propellants

`EnginePropellant` serves as an abstract collection of fuels, providing a single point to query fuel data from. It contains underlying `StoredPropellant` structs that knows about the details of each resource.

```julia
# partmodule/engine.jl

struct StoredPropellant
    name::String
    ratio::Float32
    density::Float32
    capacity::Float64
    loss::Float64
    propellant::SCR.Propellant
end

struct EnginePropellant
    resources::Vector{StoredPropellant}
    massflow::Float32
    residual_ratio::Float32
    # inner constructor
    function EnginePropellant(engine::SCR.Engine, residual_ratio::Real)
        @debug "Indexing engine propellant" _group=:index
        krpc_propellants = SCH.Propellants(engine)
        resources = Vector{StoredPropellant}()
        massflow = static_mass_flow_rate(engine)
        for kp ∈ krpc_propellants
            name = SCH.Name(kp)
            ratio = SCH.Ratio(kp)
            capacity = SCH.TotalResourceCapacity(kp)
            loss = capacity * residual_ratio
            p = StoredPropellant(name, ratio, density(name), capacity, loss, kp)
            push!(resources, p)
        end
        new(resources, massflow, residual_ratio)
    end
end
```

This way, I cached away almost all KRPC method calls, except for `SCH.TotalResourceAvailable(p)` which can change over time. In this code I also took residuals into account; some engines may be unable to burn everything and have some residuals after burnout. Here is one oddball from KRPC.jl; I cannot query a Propellant's density from the Propellant object itself; I need to ask it to the space center object. `SpaceLib` will not package the connection object with the spacecraft object, so I need to either take in `KRPC.KRPCConnection` object during propellant indexing, or have the value saved somewhere else. I took the second approach, saving the density information in `constants.jl`.

```julia
# constants.jl

const RO_DENSITY = Dict{String,Float32}(
    "Aniline22" => 1.042,
    "Hydrazine" => 1.004,
    "Hydrogen" => 8.99e-5,
    "IRFNA-III" => 1.658,
    "Kerosene" => 0.82,
    "LqdOxygen" => 1.1409999,
    "Nitrogen" => 0.001251,
    "Water" => 1.0,
    ...
)
```

I took the same approach to build abstractions over `Parachute` and `Decoupler`, but they are much more simpler than `Engine`, so I will spare the details.

[source code](https://github.com/RhahiSpace/SpaceLib.jl/blob/main/src/partmodule/engine.jl)
