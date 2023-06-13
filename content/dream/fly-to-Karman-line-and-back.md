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

[Kármán line](https://en.wikipedia.org/wiki/K%C3%A1rm%C3%A1n_line) defines altitude of 100 km as a boundary between the Earth atmosphere and space. Like many things in life, there is no such clear boundary. People proposed various altitudes for different reasons; some based on "can there be a spaceplane 'flying' at this altitude?", some based on thoughts on "lift vs. centrifugal force", and some based on "is there a possibility of having a satellite with the altitude?".

[FAI](https://www.fai.org/) defines this boundary as a well rounded number of 100 km. While this number will not serve any useful purpose in engineering, it does in contexts outside of engineering, such as law making and KSP contracts. Our mission is to program and launch a rocket that will reach at least 100 km. So how do we build a rocket that can go up that high?

### Two approaches to Kerbal rocketry

I contrast two approaches for moving a project like this forward. One approach is called *just do it*, and another may be called *work out the math*. Kerbal Space Program gives us a very nice iterative tool that lets us build fast and break things, and do it for free. This process is enjoyable, and for a simple goal like this, it is the right approach.

However, when we want to sharpen our tools and approaches, we need to also understand why the various attempts we made from *just do it* worked or not, extract out the essence, and build upon this knowledge. This process of "flying desks" is more tedious. It involves learning the science, planning the missions, and processing the data and comparing them with previous expectations.

Both approaches are important. Trying out various approaches gives us the intuition, and let us discover approaches that don't work well in its early stage. Going through textbooks and working out maths gives understanding and the precision needed to sharpen the tools we are going to make.

For this mission, I will lean on the *just do it* side, as the goal is simple and design limitation (available rocket parts) are limited enough that there is not many desicions to make.

## Series plan

### Build our first series of rockets.

I will explain the rocket's design and capabilities and reasons behind them.

### Introduce KRPC and SpaceLib

As the starting mission, I will explain the software libraries I will be using: KRPC.jl and SpaceLib.jl.

### Send it

I launch the first Karman missions and explain the code and the data I will collect from the mission.

