---
title: "first flight"
date: 2023-05-26
mathjax: false
draft: false
series: karman
tags: [sounding-rocket]
---

## Fly to the Karman line

Our mission objective is to fly a rocket to a Karman line (100 km). This is an easy mission; no re-usability, no guidance, just keep the pointy end up.

For rocket nerd's perspective, it would be quite a boring mission: Reach the Karman line. However, for a software nerd's objective, there will be some more exciting motions behind the scenes.

### Objectives

1. Reach the Karman line.
1. Log rocket's debug messages and other telemetry (altitude, speed, etc.)
1. Implement test scripts (runtests.jl) to test against regressions in library functions.
1. This is now a public project. Learn how to work cameras for some public footage and create photo/video records.

Step 3 here is the most crucial one. As I iterate and develop, the unstable codebase can suffer regression.
Testing the code reduces the mistakes and assures quality of the software.

## Mission plan

{{<image src="/images/mission/first-flight/front_Karman 1_1.png"
    alt="rocket" maxw="7em" class="right-text">}}

### Rocket

The rocket for this mission is Karman 1 and 2. They are low-tech, two-stage sounding rockets with a kick stage, statically stabilized using fins. The first stage fins are angled a few degrees to provide spin.

While having 2 liquid stages should be enough, I added a solid kick stage. It provides a powerful kick in the first second. This doesn't help much with delta-v, but shortens the time it takes for fins to spin the rocket.

### Program

The mission is to just to go up, and then blow up.

|     | Program      | Description                                        |
| --- | ------------ | -------------------------------------------------- |
|     | Test         | Check connection and actuators                     |
|     | Ignition     | Ignition & stage sequence start                    |
|     | Range safety | After the rocket starts falling, blow it up. Halt. |

- Test: issue some control commands and see if KRPC responds. Sometimes the connection is not quite right and I cannot control the rocket at all.
- Ignition: Time ignitions to maintain positive acceleration.
- Range safety: Mainly to test that this actually works. In Karman missions, the rocket burns up upon reentry anyway.

### Code

```jl
function launch(sp::Spacecraft, e1::RealEngine, e2::RealEngine)
    (@async stage1(sp, e1)) |> errormonitor
    (@async stage2(sp, e1, e2)) |> errormonitor
    stage!(sp)
    delay(sp.ts, 0.55, "SRB")
    trigger(sp, :stage1; name="launch")
end

function stage1(sp::Spacecraft, e1::RealEngine)
    waitfor(sp, :stage1)
    ignite!(sp, e1)
    delay(sp.ts, 0.2, "E1 ignition")
    stage!(sp)
    (@async stage_watchdog(sp, e1, :stage2, setevent(sp, :s1watch))) |> errormonitor
    delay(sp.ts, 47.5, "Stage 1"; interrupt=setevent(sp, :s1delay))
    trigger(sp, :s1watch; name="stage1")
    trigger(sp, :stage2; name="stage1")
end

function stage2(sp::Spacecraft, e1::RealEngine, e2::RealEngine)
    waitfor(sp, :stage2)
    trigger(sp, :s1delay; name="stage2")
    ignite!(sp, e2)
    delay(sp.ts, 0.2, "E2 ignition")
    shutdown!(e1)
    stage!(sp)
    ...
end
```

The code is written in async-wait style. First, the stage 1 and stage 2 are queued to task list. After solid stage times out, stage 1 is triggered. Stage 2 is triggered either when stage 1 times out, or when a watchdog detects that stage 1 engine is somehow turned off.

The watchdog queries the engine's state every 0.1+α seconds and then signals the next stage if anything bad happens.

```jl
function stage_watchdog(...)
    @info "$(engine.name) watchdog activated."
    while !isset(interrupt)
        if thrust(engine) == 0
            !isnothing(next) && trigger(sp, next)
            @warn "Watchdog detected engine failure" _group=:watchdog
            trigger(interrupt; name="watchdog")
            return
        end
        delay(sp.ts, 0.09; log=false)
    end
end
```

## Flight

The first flight had an engine failure, but the second stage kicked in immediately and salvaged a bit more flight time.

{{< youtube nsb4yWSy46Q >}}

---

The second flight was a success.

{{< youtube 0PVUsPdwP2I >}}

## Data

Here is the data collected by both flights. The collected data includes altitude, velocity, thrust, mass, and drag. The collected atmospheric data will be useful measure how the rocket performed in terms of aerodynamic loss versus thrust.

I forgot to include atmospheric density in the measurements, so I will include that in Karman 3.

[[karman1.data]](/data/karman1.data)
[[karman2.data]](/data/karman2.data)
