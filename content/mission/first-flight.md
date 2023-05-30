---
title: "first flight"
date: 2023-05-26
mathjax: false
draft: false
series: karman
tags: [sounding-rocket]
---

## Fly to the Karman line

Our mission objective is to fly a rocket to the Karman line (100 km). This is an easy mission; no re-usability, no guidance, just keep the pointy end up.

For a rocket nerd's perspective, it would be quite a boring mission. However, for a software nerd's objective, there will be some more exciting motions behind the scenes.

### Objectives

1. Reach the Karman line.
1. Log the rocket's debug messages and other telemetry (altitude, speed, etc.)
1. Implement test scripts (runtests.jl) to test against regressions in library functions.
1.Learn how to work cameras for some public footage and create photo/video records.

Step 3 here is the most crucial one. I do not know if the functions I create are going to work well. As I run the rockets, I will have to go back and fix them. In this unstable code base, regression is common -- features that worked in the past might break later. To remedy this situation, I need to test my code and do it often.

## Mission plan

{{<image src="/images/mission/first-flight/front_Karman 1_1.png"
    alt="rocket" maxw="7em" class="right-text">}}

### Rocket

The rockets for this mission are [Karman 1 and 2](/craft/karman-2). They are low-tech, two-stage sounding rockets with a kick stage, statically stabilized using fins. The first stage fins are angled a few degrees to provide spin.

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

Here is the simplified version of the mission code. The actual code has a bit more supporting structures for signaling between stages.

```jl
function launch(sp::Spacecraft)
    stage!(sp)
    delay(sp.ts, 0.55, "SRB")
    notify(...)
end

function stage1(sp::Spacecraft, e1::RealEngine)
    wait(...)
    ignite!(sp, e1)
    delay(sp.ts, 0.2, "E1 ignition")
    stage!(sp)
    @async stage_watchdog(...)
    delay(sp.ts, 47.5, "Stage 1")
    notify(...)
end

function stage2(sp::Spacecraft, e1::RealEngine, e2::RealEngine)
    wait(...)
    ignite!(sp, e2)
    delay(sp.ts, 0.2, "E2 ignition")
    shutdown!(e1)
    stage!(sp)
    ...
end
```

The code is written in an async-await style. First, the stage 1 and stage 2 are queued to the task list. After solid stage times out, stage 1 is triggered. Stage 2 is triggered either when stage 1 times out, or when the watchdog started in stage 1 detects engine flameout.

## Flight

The first flight had an engine failure, but the second stage kicked in immediately and salvaged a bit more flight time.

{{< youtube nsb4yWSy46Q >}}

---

The second flight was a success.

{{< youtube 0PVUsPdwP2I >}}

## Data

Here is the data collected by both flights. The collected data includes altitude (m), velocity (m/s), thrust (N), mass (kg), and drag (N). The collected atmospheric data will be useful measure how the rocket performed in terms of aerodynamic loss versus thrust.

I forgot to include atmospheric density in the measurements, so I will include that in Karman 3.

[[karman1.data]](/data/karman1.data)
[[karman2.data]](/data/karman2.data)
