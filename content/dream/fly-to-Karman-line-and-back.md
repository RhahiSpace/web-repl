---
title: "fly to Karman line and back"
date: 2023-06-11
mathjax: false
draft: false
series: fly to Karman line and back
tags: []
---

{{<quote text="The goal of peaceful space exploration starts here, with rocketry missions that perform key experiments ... [and] handle brief exposures to space and the rigors of rocket travel." class="center" who="RP-1">}}

## Introduction

### Karman line at the edge of space

[Kármán line](https://en.wikipedia.org/wiki/K%C3%A1rm%C3%A1n_line) defines altitude of 100 km as a boundary between the Earth atmosphere and space. Like many things in life, there is no such clear boundary between the Earth atmosphere and space. People proposed various altitudes for various reasons, some based on "can there be a spaceplane 'flying' at this altitude?", others based on thoughts on "lift or centrifugal force, which is more dominant?", and another based on "is it possible to orbit a satellite with this altitude?".

[FAI](https://www.fai.org/) defines this boundary as a well rounded number of 100 km. While this number will not serve any useful purpose in engineering, it does in context outside of engineering, such as law making, or in KSP contarcts. So, our mission is to program and launch a rocket that will reach at least 100 km. So how do we build a rocket that can go up that high?

### Two approaches to Kerbal rocketry

I contrast two approaches for moving a project like this forward. One approach is called *just do it*, and another may be called *work out the math*. Kerbal Space Program gives us a very nice iterative tool that lets us build fast and break things, for free. This process is enjoyable, and for a simple goal like this, it is the right approach.

However, when we want to iterate and sharpen the approaches, we need to also understand why the various attempts we made from *just do it* worked, extract out the essence of it, and build upon this knowledge. This process of "flying desks" is more tedious. It involves learning the science, planning the missions, and processing the data and comparing them with previous expectations.

Both approaches are important. Trying out various approaches out gives us intuition, and let us prune out approaches that don't work well at early stages. Going through textbooks and working out maths gives understanding, and provides the precision needed to sharpen the tools we are going to make.

For this mission, I will lean on the *just do it* side, as the goal is simple and design limitation (available rocket parts) are limited enough that there is not much desicion space.

## Series plan

### Build our first series of rockets.

I will explain the rocket's design and capabilities and reasons behind them.

### Introduce KRPC and SpaceLib

As the starting mission, I will explain the software libraries I will be using: KRPC.jl and SpaceLib.jl.

### Send it

I launch the first Karman missions and explain the code and the data I will collect from the mission.

