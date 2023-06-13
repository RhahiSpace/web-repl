---
title: "KRPC"
date: 2023-06-12T21:41:00+02:00
mathjax: false
draft: false
series: fly to Karman line and back
tags: []
---

## Introduction

Before I start explaining some code, it will be good to explain what is KRPC and how to use it.


{{<image src="https://ksp-kos.github.io/KOS/_images/hello_world1.png" class="right" maxw="15em" title="kOS in-game terminal">}}


There are two notable projects that lets you control KSP spacecraft with your own program. [KRPC](https://krpc.github.io/krpc/) lets you do it using programs running outside of the game, using various programmin languages. [kOS](https://ksp-kos.github.io/KOS/) implements its own programming language called Kerboscript and runs inside the game. The main advantage of kOS is that it robust and synchronizes with the physics very nicely. You can guarantee that your control loops will work with the right timing, so it is perfect for [real time applications](https://en.wikipedia.org/wiki/Real-time_operating_system). However, it is limited to its own programming language, it lacks the expanse of libraries that other communities have accomulated over time, and you can't easily pick a style that fits well with your domain.

I will not be programming in kOS, but I keep it in mind as an option if a real-time control requirement becomes strict enough.

KRPC officially supports C-nano, C#, C++, Java, Lua, and Python. However, there are independent projects that expands KRPC for languages like Julia, Rust, and others.

## KRPC.jl

[KRPC.jl](https://github.com/BenChung/KRPC.jl/) is a KRPC client for Julia. It has more bugs compared to more stable client like Python, but I like to think that it has become reliable enough to use these days. I am also biased, becuase I've been contributing bug fixes to it!

Here is a quick tutorial on how to use KRPC.jl to install, connect, take control, and look up the documentations.

### Install

KRPC is registered in general Julia registry, so you can simply add it using `] add`. You may want to work with latest master, then use `] add KRPC#master`.

### Some assembly required

KRPC is not a 100% dynamic client, so you need to generate code before using. This is done by using `generate_stubs` function. If you also `] dev KRPC` as well, you can pass `save_file=true` flag to save stubs directly into the source code.

```julia
using KRPC

conn = kerbal_connect("MyKRPC", "127.0.0.1")
# the game might freeze while trying to connect.
generate_stubs(conn; save_file=true)
# the code will be generated in your local source code.
```

After this, restart Julia and you can now use KRPC with its full capabilities.

### Documentations included, soft of.

Because KRPC is mostly documented in the official website, you can look at those for most documentation problems. Within Julia, there are a few functions that will also help you with navigationg through the API.

#### methods

use `methods` to learn about what methods are available sharing the same function.

```julia
import KRPC.Interface.SpaceCenter.Helpers as SCH
SCH.Active |> methods
# 8 methods for generic function "Active" from KRPC.Interface.SpaceCenter.Helpers:
 [1] Active(this::KRPC.Interface.SpaceCenter.RemoteTypes.Contract)
     @ ~/.julia/dev/KRPC/src/generated.jl:11294
 [2] Active(this::KRPC.Interface.SpaceCenter.RemoteTypes.Engine)
     @ ~/.julia/dev/KRPC/src/generated.jl:13128
 [3] Active(this::KRPC.Interface.SpaceCenter.RemoteTypes.Light)
     @ ~/.julia/dev/KRPC/src/generated.jl:13594
...
```

#### methodswith

use `methodswith` to figure out the use cases of a type within a module.

```julia
import KRPC.Interface.SpaceCenter as SCR
import KRPC.Interface.SpaceCenter.Helpers as SCH
methodswith(SCR.Part, SCH)
```

#### names

use `names` to explore what kind of structs, constants, functions, etc are defined within a module..

```julia
import KRPC.Interface.SpaceCenter.Helpers as SCH

vec = names(SCH)
ioc = IOContext(stdout, :limit => false)
show(ioc, vec)
# a long list appers! Save it somewhere as a file to use Ctrl+F later.
```

#### documentation

I prefer to use IDE/editor's language server instead of using REPL to find out docstrings, because I was not able to look up specific method's docstrings.

```julia
help?> SCH.Active!
Active!(this::RemoteTypes.Sensor, value::Bool)

Whether the sensor is active.
```

#### noteworthy modules

Here are modules that contain structs and functions that seem useful with the abbreviations that I like to use:

```julia
# central modules
import KRPC.Interface.SpaceCenter as SC
import KRPC.Interface.SpaceCenter.RemoteTypes as SCR
import KRPC.Interface.SpaceCenter.Helpers as SCH

# uncommonly used central modules
import KRPC.Interface.KRPC as KK
import KRPC.Interface.KRPC.RemoteTypes as KR
import KRPC.Interface.KRPC.Helpers as KH

# UI related
import KRPC.Interface.UI as UI
import KRPC.Interface.UI.RemoteTypes as UIR
import KRPC.Interface.UI.Helpers as UIH

# Drawing related
import KRPC.Interface.Drawing as DD
import KRPC.Interface.Drawing.RemoteTypes as DR
import KRPC.Interface.Drawing.Helpers as DH

# some mods can also expose methods, and this will be the primary way to discovery them.
import KRPC.Interface.MechJeb as MJ
import KRPC.Interface.MechJeb.RemoteTypes as MJR
import KRPC.Interface.MechJeb.Helpers as MJH
```

Note that three modules are always bundled together. RemoteTypes is a collection of types, useful for strongly typing your functions. Helpers are the functions that you normally call to fetch or set values. The parent module is used when creating KRPC streams or possibly events.

### Example code
```julia
using KRPC
import KRPC.Interface.SpaceCenter.Helpers as SCH

# setup connections and objects
conn = kerbal_connect("MyKRPC", "127.0.0.1")
center = SCR.SpaceCenter(conn)
ves = SCH.ActiveVessel(center)
control = SCH.Control(ves)

# address commands, note the type.
SCH.Throttle!(control, 1.0f0)
SCH.ActivateNextStage(control)

# fetch current time, vessel's active thrust, etc
@show SCH.UT(center)
@show SCH.Thrust(ves)
```

These are merely building blocks, and it is up to us to build something great.

## SpaceLib.jl

SpaceLib is my personal project of creating an abstraction layer on top of KRPC.jl. It has following goals:

1. Make it easy to set up connections, add spacecrafts, and actuate them.
1. Abstract complicated part interaction, control loops, and communications away for concise mission code.
1. Enable interactive control of an ongoing mission, without having to restart the entire program.

I will also be developing several other software packages along the way to aid this effort, such as logging, trajectory generation, and user interfaces. This code base will be largely unstable until announced, so I do not recommend you to use it and rely on it yet.

Here is what my typical script might look like with SpaceLib.

```julia
# setup
center = SpaceCenter("Karman", "10.0.0.51")
sp = add_active_vessel!(center)
parts = SCH.Parts(sp.ves)
con = subcontrol(sp, "MyControl")
core = SCH.WithTag(parts, "core")[1] |> CommandModule.ProbeCore
e1 = SCH.WithTag(parts, "e1")[1] |> Engine.RealEngine

# functions
function stage0()
    stage!(sp)
    delay(sp.ts, 0.65, "SRB")
end

function stage1()
    ignite!(sp, e1; name="E1 ignition")
    stage!(sp)
    wait_for_burnout(sp, e1;
        progress=true,
        name="Stage 1",
        margin=0,
        timeout=50,
        interrupt=event!(sp, :s1),
        allow_engine_shutoff=false,
    )
end

# calls
stage0()
stage1()
```

And here is a little demo of what a mission might look like.

{{<youtube ATAYY8V287U>}}

