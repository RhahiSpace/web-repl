---
title: "Karman 1"
date: 2023-06-12
mathjax: false
draft: false
series: fly to Karman line and back
tags: []
---

*For craft files and specs, go to [Karman Rockets](/craft/karman)*

---

{{<image src="/craftfiles/Karman%201.png" class="right" maxw="35%">}}

Mission requirements directly influence the design of rockets. Our mission is to fly the rocket to at least 100 km and back. However, there are many more trade-offs and design choices to make.

Let's go over each consideration I made.

## "Large, or extra large?"

Karman 1 will be overpowered for the 100 km objective. This is a deliberate decision based on following factors:

### The launch complex

In RP-1, rockets are built using [launch complexes](https://github.com/KSP-RO/RP-1/wiki/Launch-Complexes-and-Programs-Conversion-Guide). I decided to have *Karman Complex* support launch mass support 1-2 tons. It was not possible to set fractional weight. With exception of Karman 1 mission, all Karman rockets are likely be at least 1-ton heavy, so I thought this will be more economical, with better range to upscale the launch complex.

A 100 km sounding rocket can be achieved in less than 1 tons. However, since the launch complex will not support such rockets, I must build an overpowered rocket for the 100 km objective. This extra cost is not a problem becuase the Karman rockets are small and cheap.

### High altitude missions

Launching a rocket with higher altitude, downrange, or payload mass will require me to build larger rockets. Along with having larger launch compelx, designing Karman 1 with these extra ranges in mind will make it easier to use for future missions.

### Tooling costs

Another reason to have a larger rocket body is [tooling](https://github.com/KSP-RO/RP-1/wiki/What-is-tooling%3F) cost. I am using [WAC Corporal](https://en.wikipedia.org/wiki/WAC_Corporal#Specifications) rocket motors. It is not a very powerful rocket, so I will not consume as much fuel as the next variant, XASR-1. With XASR-1, I will need a larger fuel tank to burn for the full duration. Therefore, I designed the stage with XASR-1 in mind, leaving some empty spaces for Karman 1.

## General rocket design

- I have chosen 400 mm as the main body diameter, because it is a nice size to work with and is on par with the engine diameter.

- The nose cone is an empty tank, capable of holding 95 units of sounding payload.

- The fins are placed in each stage, in the lowest point to act as passive stabilizers. I did not put in tilt in the fins for spin stabilization.

    Oddly, adding tilted fins caused worsening precession and caused loss of attitude in the end. I did not experience this in my previous iterations, and I do not know the reason yet.

- The decouplers will not exert extra force upon separation, as that can kick the next stage into bad attitude.
