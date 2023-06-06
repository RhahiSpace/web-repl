---
title: "troubleshooting KRPC.jl"
date: 2023-06-06
mathjax: false
draft: false
series:
tags: []
---

{{<quote text="Houston, we've had a problem." class="white center" who="13">}}

I have been programming in Julia and KRPC.jl for about a year now. KRPC.jl is a small independent project created by one person, and it is not without bugs. Some of them is not very important -- it's not mission-critical that some enumerations have weird numbers, nor that decoupling a vanilla separator will raise a recoverable exception.

Yesterday, I hit a mission critical bug.

## The problem

I've been implementing a clustered engine API. You might have read how I implemented Engine API. This API works for a few engines, but when I try to fetch about 5 engines, then I get irrecoverable error from KRPC.

What do I mean by irrecoverable? It means that something in the client or the server got messed up, and it will no longer respond correctly to future KRPC calls. The only possible recovery is to restart the server, which is done by hand. I've been aware of this problem, but it only happened when I re-wind and use the same connection for the next launch. Not a big deal. I restart the client and everything is ok. One day I might actually make an automation that presses that stop/start button for me!

## The solution

I see 4 outs to this problem. Let's discuss them one by one.

1. Fix KRPC.jl
1. Use PyCall to call python client
1. Program in Python
1. Program in Rust

### Fix KRPC.jl

This has been my go-to-approach. I spent a whole day trying to identify and fix the bug, but I was not able to find the problem. Maybe I should spend more time to fix? For now, I am waiting for the author to respond to the [issue](https://github.com/BenChung/KRPC.jl/issues/16).

### Use Pycall

[PyCall](https://github.com/JuliaPy/PyCall.jl) is a library that enables using Python within Julia.

This mostly works, with some disadvantages.
- Type information is lost. To get type information, I need to create a wrapper for each type (this is what KRPC.jl does). I cannot really verify that the correct object was assigned, but as long as my library code is correct and tested all subsequent calls will be typed.
- Cannot stream. Along with the lost type information, the information about the state of stream calls are also lost. So I cannot natively retrieve streamed values and push them into Julia channels --- this is the major drawback when resorting to PyCall.

### Program in Python

I do speak Python, but writing SpaceLib in Python is not something I am looking forward to at the moment.

### Program in Rust

This is the spicy option. Part of the reason why I picked Julia was that I have access to various high-performant numerical software suite at hand, with good prototyping speed and without the high learning curve of Rust. Maybe it is a great opportunity, still. After all, programming in KSP is how I learned programming in general. I am a bit concerned about the maturity and buggy-ness of the Rust crate for KRPC, though, so I will ask around.

The problem with Rust program client is that I will lose the ability to interactively control the rocket. Not a big issue, but it's one feature lost.

## The fix

I spent about 8 hours trying to troubleshoot. Stepping through the code, trying to sniff the signals with Wireshark (I couldn't figure that out), just staring at the code, and sleeping over it. Each attempt and failure got me more accustomed to the client code, and finally I understood what went wrong.

The problem was that when KRPC.jl generates a new request, the request gets an ID. When the ID reaches 128, KRPC.jl breaks down, and no further new request can be made. (I can still reuse the old ones). 128 is an important number here, because it is half of 256, size of a `UInt8`. During the "try and fail" phases, I learned that ProtoBuf encodes the given data into arrays of `UInt8`s. While looking at some of ProtoBuf's code to convert numbers, I found a method

`write_varint` and `write_fixed`. `write_varint` splits given integer data into 7-bit chunks, with the first bit representing end of data or continuation of data. KRPC.jl was using `write_fixed` to convert its data, which does not follow the 7-bit rule. Therefore, when the ID reaches 128, The first bit flips to 1, saying "the next byte is also a data". This is not true, and causes the KRPC server to misinterpret the ID and I get wrong objects back. Most of the case, this will be an object of the wrong type, and I get a type related error.

The fix is to change the method `KRPC.getWireValue` for integers to use `write_varint`. This may introduce a bug, because there might be other cases where it must use the fixed-integer form. Then, I will either have write a handler for the Request dispatch to identify when the field is an ID and only use `write_varint`.

But for now, the problem is solved, and I can continue developing the cluster engine.
