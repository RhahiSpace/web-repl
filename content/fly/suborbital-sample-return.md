---
title: "suborbital/sample return"
date: 2023-05-30
mathjax: false
draft: false
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

In the previous mission, I used time-based delay to stage engines with a watchdog to do emergency staging if something went wrong. This time, I removed the watchdog and staged based on remaining fuel and the engine status. This lets us keep running the rocket as long as possible and utilize overburn if needed. To read about how I implemented the engine API, see [engine abstraction and caching](/apply/engine-abstraction)

## Interactive rocket control

Ever since I started *Kerbal Space Programming*, I wanted to do full automated missions. Sometimes, though, it's nice to be able to pause the program in the middle without stopping the program entirely.

I see two approaches to achieve this: [Jupyter](https://en.wikipedia.org/wiki/Project_Jupyter) and [REPL](https://docs.julialang.org/en/v1/stdlib/REPL/). Jupyter+Julia in VSCode has a bug that makes it less great compared to Jupyter+Python, but it still makes interactive development much easier. I also use REPL, but in this mission I limited myself to Jupyter to have a more focused mission.

### The Jupyter bug

The bug is that when interrupting or stopping kernel, the UI will keep going. The cell does stop, but the UI runs forever, blocking further execution. This is a problem when I want to run some busy loop and then stop it to do something else. If I run the loop in an async block, however, the cell will stop as soon as the task is declared, with the scheduled task running in the background. This is exactly what I am going to do; run every maneuver in async, and then somehow control them via Julia's  signaling mechanisms.

### The signaling mechanism

Here is a stage1 code of Karman 5 mission as an example.

I first ignite the rocket engine, and then do a stage separation. The `wait_for_burnout` function will constantly check the remaining fuel and return if the fuel runs out or stops changing (indicating engine burnout).

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

The past parameter contains `event!(sp, :s1)`. In SpaceLib, I made a struct called `EventCondition`, a combination of `Event` and `Condition`. Event is a level-triggered source, where once you `set` an event it will stay set and all other tasks can see it set. `Condition` is an edge-triggered source, where only the tasks that are waiting on it will get notified when the condition is triggered.

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

Wouldn't it be better if I just queue two stage 2 variants; one with regular maneuver, and another with ASAP maneuver, and trigger only the one I want? I don't know. I will figure it out when I get there!

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

While carrying biological payload is not a requirement in Karman 3, I added it to collect "science" earlier (which is an in-game resource to unlock better technology). Due to the added payload, Karman 1 rocket with the same payload will not reach 140 km.

In Karman 5, I used the science gained to unlock an improved rocket engine, XASR-1. Thanks to the improved performance, I no longer need the double kick stage.

### Program

| Program              | Description                                   |
| -------------------- | --------------------------------------------- |
| Test                 | Test connections and index parts              |
| Ignition             | Ignition & stage sequence start               |
| Core separation      | Separate nose cone and rocket at the apoapsis |
| Parachute deployment | -                                             |

Since this is a return mission, there is no range safety step. Instead, the core is separated and the parachutes are armed, so that the small mammals can return home safely.

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
# Karman5.ipynb

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

If for some reason I want to stop stage 1 immediately and jump to stage 2, I can open up another cell and run code `notify(sp, :s1)` so that `wait_for_burnout` function will break its loop and return.

When the code is running this way, I cannot monitor errors using `errormonitor`. Instead, I made the code notify if there is an error, so I can check it by `wait`ing on the task.

[Source code](https://github.com/RhahiSpace/MissionLib.jl/tree/main/Karman3)

## Flight

All 3 flights were successful. Tried different filming angles for each of them. I like that Karman 5 launch gives the feeling of ground-based camera station, so I might do more of that in the future.

{{<youtube ATAYY8V287U>}}

### Data

In Karman 3, I wrote a wrong call for data collection and no data was collected. In Karman 4, this bug was fixed and you can see the data [here](/data/karman4.data). no data measurement was done for Karman 5.
