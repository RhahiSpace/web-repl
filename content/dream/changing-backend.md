---
title: "changing the backend"
date: 2023-06-06
mathjax: false
draft: false
series:
tags: []
---

{{<quote text="Houston, we've had a problem." class="white center" who="13">}}

I started programming in Julia for about a year now, and ever since I have been playing KSP with Julia. KRPC.jl is a small independent project created by one person, and it is not without bugs. Some of them is not very critical -- it's not mission-critical that some enumerations have odd numbers, nor that decoupling a vanilla separator will raise a recoverable exception.

Yesterday, I hit a mission critical bug.

## The problem

I've been writing some logic for grouped control of clustered engines. You might have read how I implemented Engine API. This API works for a few engines, but when I try to fetch about 5 engines, then I get irrecoverable error from KRPC.

What do I mean by irrecoverable? It means that something in client or server got messed up, and it will no longer respond correctly to future KRPC calls. The only possible recovery is to restart the server. This action cannot be done via KRPC, so it should either be done by hand or some kind of image detection macro. I've been aware of this kind of problem, but it only happened when I re-wind and try to keep using the same connection for the next launch. So not a big deal. One day I might actually make an automation that presses that stop/start button for me from Julia!

## The solution

I see 4 outs to this problem. Let's discuss them one by one.

1. Fix KRPC.jl
1. Use PyCall to call python client
1. Program in Python
1. Program in Rust

### Fix KRPC.jl

This has been my go-to-approach. I spent whole day trying to identify the bug troubleshoot, but I was not able to fix it. Maybe I should spend more time to fix? For now, I am waiting for the author to respond to the [issue](https://github.com/BenChung/KRPC.jl/issues/16).

### Use Pycall

[PyCall](https://github.com/JuliaPy/PyCall.jl) is a library that enables using Python within Julia.

This mostly works, with some disadvantages.
- Type information is lost. To get type information, I need to create a wrapper for each type (this is what KRPC.jl does). I cannot really verify that the correct object was assigned, but as long as my library code is correct and tested all subsequent calls will be typed.
- Cannot stream. Along with the lost type information, the information about the state of stream calls are also lost. So I cannot natively retrieve streamed values and push them into Julia channels --- this is the major drawback when resorting to PyCall.

### Program in Python

I do speak Python, but writing SpaceLib in Python is not something I am looking forward to at the moment.

### Program in Rust

This is the spicy option. Part of the reason why I picked Julia was that I have access to various high-performant numerical software suite at hand, with good prototyping speed and without the high learning curve of Rust. Maybe it is a great opportunity, still. After all, programming in KSP is how I learned programming in general. I am a bit concerned about the maturity and buggy-ness of the Rust crate for KRPC, though, so I will ask around.

## The conclusion?

Not decided. Come back later.
