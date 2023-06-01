---
title: "suborbital/sample return"
date: 2023-05-30
mathjax: false
draft: true
series: karman
tags: [mission, suborbital]
---

## Fly to space, and come back safely

Last time, we touched the edge of space and blew up. This time, we will come back instead of blowing up.

### Objectives

1. Reach space and return home with the sample payload.
1. Record the atmospheric density data.
1. Implement API for engine, parachute, and decoupler.
1. Write an interactive mission program, not a script.

## API for part/modules

In the previous mission, I used time-based delay to stage engines, with a watchdog to do emergency staging if something went wrong. This time, I am going to stage engines based on remaining fuel and the engine's active status. To read about how I implemented the engine API, see [engine abstraction and caching](/apply/engine-abstraction)

## Interactive rocket control

Long since I started *Kerbal Space Programming*, I wanted to have full automated missions, while keeping the ability to pause the program an re-program the rocket as needed without stopping the program entirely.

I see two approaches to this: [Jupyter](https://en.wikipedia.org/wiki/Project_Jupyter) and [REPL](https://docs.julialang.org/en/v1/stdlib/REPL/). Jupyter in VSCode has some limitations (or bugs) that makes it less great compared to Jupyter+Python in VSCode, but it still makes interactive development much easier. I also use REPL, but in this mission I limited myself to Jupyter to have a more focused mission.

### The Jupyter bug

Jupyter notebook in VSCode has a bug where interrupting or stopping kernel will not stop the UI. The cell does stop, but the UI will run forever, blocking further execution. This is a problem when I want to run some busy loop and then stop it to start over. If I run the loop in an async block, however, the cell will stop as soon as the task is declared, with the scheduled task running in the background. This is exactly what I am going to do; run every maneuver in async, and then somehow control them via Julia's  signaling mechanisms.

### The signaling mechanism

Take a look at stage 1 part of Karman 5 mission.

I first ignite the rocket engine, and then have a stage separation. The `wait_for_burnout` function will constantly check the remaining fuel and return if the fuel runs out or stops changing (indicating engine burnout).

```julia
# Karman5.ipynb

function stage1(sp::Spacecraft)
    ignite!(sp, e1; name="E1 ignition")
    stage!(sp)
    wait_for_burnout(sp, e1;
        progress=true,
        name="Stage 1",
        margin=0,
        timeout=50,
        interrupt=event!(sp, :s1),
    )
end
```

The past parameter contains `event!(sp, :s1)`. In SpaceLib, there is a struct called `EventCondition`, which is a combination of `Event` and `Condition` of Julia. Event is a level-triggered source, where once you `set` an event it will stay set and all other tasks can see it set. `Condition` is an edge-triggered source, where only the tasks that are watching it will know when the condition is notified, and future monitors will not see the trigger.

```julia
struct EventCondition
    cond::Condition
    active::Ref{Bool}
    value::Ref{Any}
    function EventCondition(active::Bool=false, value::Any=nothing)
        new(Condition(), active, value)
    end
end
```

When a task `wait`s on `EventCondition`, first it checks the `active` flag and if it is set, it will use the stored `value` immediately, without waiting. If the flag has not been set, then it will start waiting for its `Condition`, which will return `value` when it gets notified, breaking the wait.

Here are the reasons on why I want to combine the two mechanisms together:

1. Level-trigger makes programming easier. I don't need to worry about making sure that the task already started listening before I fire the event.
1. `Event` cannot transport extra data like `Condition` does; for example, if I trigger a stage 2 of the rocket, I might want to tell *"hey, the previous stage had a problem, so you should skip the alignment maneuver and just fire up engine ASAP"*.

Wouldn't it be better if I just queue two stage 2 variants; one with regular maneuver, and another with ASAP maneuver, and trigger only the one I want? I don't know. I will figure it out when we get there!

*Note: I see one potential issue where a task can get stuck if the event has been triggered right after checking `active` value. It can be remedied by using a semaphore or somehow forcing atomic operation, but I keep it as-is for now.*

## Mission plan

{{<image src="/craftfiles/Karman 3.png"
    alt="rocket" maxw="7em" class="right-text">}}

There are several of sample return missions, and they are mostly similar barring the changing payload and altitude requirements.

- Karman 3: no payload required, 140 km and return.
- Karman 4: a small biological capsule, 35 units of sounding rocket payload, 100 km and return.
- Karman 5: a small biological capsule, 75 units of sounding rocket payload, 120 km and return.

The "small biological capsule" is supposedly holding small mammals to study the effects of spaceflight on life. No real or virtual mammals were harmed during the experiment. The "sounding rocket payload" represents payload that scientists have designed for various high altitude experiments, and the rocket will carry the required amount in the nose cone.

### Rocket

[Craft files](/craft/karman)

While carrying biological payload is not a requirement in Karman 3, I added it to collect "science" earlier (which is an in-game resource to unlock better technology). Due to the added payload, Karman 1 design is not enough to reach 140 km.

In Karman 5, I used the science gained to unlock an improved rocket engine, XASR-1. In real life, this engine was used for [Aerojet General X-8](https://en.wikipedia.org/wiki/Aerojet_General_X-8).

### Program

| Program              | Description                                   |
| -------------------- | --------------------------------------------- |
| Test                 | Test connections and index parts              |
| Ignition             | Ignition & stage sequence start               |
| Core separation      | Separate nose cone and rocket at the apoapsis |
| Parachute deployment | -                                             |

Since this is a return mission, there is no range safety step and instead we separate the core and arm parachutes, so that the small mammals can return home safely.

### Code

With the addition of Engine/Parachute/Decoupler API, the code looks much nicer! This code comes right from Karman 5 mission, without any clean-up for the blog post.

```julia
# Karman5.ipynb

function stage0(sp::Spacecraft)
    stage!(sp)
    delay(sp.ts, 0.6, "SRB")
end

function stage1(sp::Spacecraft)
    ignite!(sp, e1; name="E1 ignition")
    stage!(sp)
    wait_for_burnout(sp, e1;
        progress=true,
        name="Stage 1",
        margin=0,
        timeout=50,
        interrupt=event!(sp, :s1),
    )
end

function stage2(sp::Spacecraft)
    ignite!(sp, e2; name="E2 ignition")
    shutdown!(e1)
    stage!(sp)
    wait_for_burnout(sp, e2;
        progress=true,
        name="Stage 2",
        margin=0,
        timeout=50,
        interrupt=event!(sp, :s2),
    )
end

function deploy(sp::Spacecraft)
    event(sp, :s3; create=true)
    periodic_subscribe(sp.ts, 5) do clock
        ref = ReferenceFrame.BCBF(sp.ves)
        h0 = SCH.Position(sp.ves, ref) |> norm
        h_prev = h0
        for now = clock
            isset(sp, :s3) && break
            h = SCH.Position(sp.ves, ref) |> norm
            h - h_prev < 0 && break
            h_prev = h
            yield()
        end
    end
    arm!(chute)
    delay(sp.ts, 1)
    if isarmed(chute)
        @info "Parachute has been armed" _group=:module
    end
end

function detach(sp::Spacecraft)
    decouple!(dec1; top=true)
    decouple!(dec2; bottom=true)
end
```

To make this code interactable, I wrapped the execution block in async.

```julia
@async begin
    try
        stage0(sp)
        stage1(sp)
        stage2(sp)
        deploy(sp)
        detach(sp)
    catch e
        @error "Error has occured, to review the error, wait on this task." _group=:system
        error(e)
    end
end
```

When the code is running this way, I cannot monitor errors using `errormonitor`. Instead, I made the code throw an error and I can check it by `wait`ing on the task.

So, if for some reason I want to stop stage 1 immediately and jump to stage 2, I can open up another cell and run code

`notify(sp, :s1)`

so that `wait_for_burnout` function will break its loop and return.

[Source code](https://github.com/RhahiSpace/MissionLib.jl/tree/main/Karman3)

## Flight

All 3 flights were successful. Tried different filming angles for each of them. I like that Karman 5 launch gives the feeling of ground-based camera station, so I might do more of that in the future.

{{<youtube ATAYY8V287U>}}

### Data

In Karman 3, I wrote a wrong call for data collection and no data was collected. In Karman 4, this bug was fixed and you can see the data [here](/data/karman4.data). no data measurement was done for Karman 5.
