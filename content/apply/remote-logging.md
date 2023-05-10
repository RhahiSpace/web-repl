---
title: "how RemoteLogging.jl works"
date: 2023-05-10T00:01:40+02:00
mathjax: false
draft: false
series: telemetry
tags: [julia, libraries]
---

> Warning: This post is not beginner-friendly, and require experience with Julia, ProgressLogging and LoggingExtras.


RemoteLogging library is about sending log messages over the network. For context, it is not about sending logs over log aggregators with a nice looking dashboard. It is simply detaching my main program (where the REPL sits) from major stream of logs. Otherwise, I will get spammed by the logs as soon as I do *anything*!

Like many of my projects, the first implementation was idiotic, awkward, and inefficient. That was last year. My Julia skills haven't improved much since, but I have fresh eyes and new ideas.

## The business logic

The Julia `Sockets` library makes network IO very easy. All I need to do is get a connection, and plug that TCP connection into a `println` function. When using `LoggingExtras.jl` library, this part is handled when I set up a logger with custom IO.

```jl
tcp = connect(host, port)
logger = TerminalLogger(tcp)
```

This works, but it loses color information in the other side. To fix it, I need to supply IOContext as well.

```jl
tcp = connect(host, port)
dsize = (displaysize()[1], displaywidth)
ioc = IOContext(tcp, :color => true, :displaysize => dsize)
logger = TerminalLogger(ioc)
```

And I have colors.

Since this code is sending logs, I will call it a *talker*. On the other side, I need a *listener* that creates a server and waits for a talker to connect to. That can be done with `listen` and `accept`.

```jl
server = listen(host, port)
conn = accept(server)
data = String(readline(conn))
println(stderr, data)
```

And that is the fundamentals! Next, need to wrap it in a struct so that I can use it just like any other logger.

## Wrap it

To make Julia understand that the struct I made is a logger, it needs to be structured like this:

```jl
struct RemoteLogger{T<:AbstractLogger} <: AbstractLogger
    tcp1::TCPSocket
    tcp2::TCPSocket
    logger::T
end
```

Since I have TCP connections going on, I also added additional fields for that. I added two TCP fields here. More on that later.

Just trying to log using this logger produces errors. This is because there are no implementations that use my new `RemoteLogger` yet. Seeing the error message, reading what the method is about, and implementing one by one, I ended up adding 4 more supporting methods.

```jl
shouldlog(_::RemoteLogger, args...) = true
min_enabled_level(_::RemoteLogger) = BelowMinLevel
catch_exceptions(logger::RemoteLogger) = catch_exceptions(logger.logger)
handle_message(logger::RemoteLogger, args...; kwargs...) = handle_message(logger.logger, args...; kwargs...)
```

- `shouldlog` and `min_enabled_level` is a simple *allow-all* case. The actual log handing will be implemented using compositional loggers in `LoggingExtras.jl`.
- `catch_exceptions` marks that any errors produced with @logmsg macros will be ignored. I don't want rockets to halt because a logger broke.
- In `handle_message` I simply defer to underlying logger. Since `LoggingExtras.jl` already implements `handle_message` for their own logger, I can get it for free.

## What about progress?

One feature I would love to add is also `ProgressLogging.jl`. In my rocket program, when I am keeping track of remaining time until next event (i.e., stage separation, reaching certain atmosphere, or just waiting for a few more seconds to reach full thrust) I would like to visualize that. ProgressLogging is a neat way to do this.

Using above implementation, we don't get progress logs, we just get a message like `Progress: 10%`, instead of an actual progress bar. This is because I am merely printing the rendered content. To fix it, I need to receive `Progress` struct and then re-play it on the listener.

### Sending a struct

A struct can also be sent over TCP as long as the struct can be serialized. Since `Progress` can be serialized, there is no additional step to process it other than digging its data and sending it over the network.

```jl
function get_progress_logger(tcp)
    logger = TerminalLogger(devnull)
    logger = TransformerLogger(logger) do log
        serialize(tcp, log.message.progress)
        return log
    end
    logger = EarlyFilteredLogger(logger) do log
        log.group == :ProgressLogging
    end
    return logger
end
```

In this code, and incoming message is first filtered by checking its group. ProgressLogging adds log group `:ProgressLogging` into its log messages, so I can filter using that. This saves me a lot of trouble and some computing time. After serializing and sending it over the network, I discard the message into devnull.

Then, the listener shows the progress message like regular log messages, but with @info macro instead of `println`.

```jl
server = listen(host, port)
conn = accept(server)
logger = TerminalLogger()
with_logger(logger) do
    msg = deserialize(conn)
    @info ProgressLogging.ProgressString(msg)
end
```

![logging](/images/library/logging-example.png)

### Cleaning up

There is a problem, though. When using ProgressLogging over remote, ending a speaker in the middle of ongoing progress will cause the progress bar to get stuck. This is not very pleasant because simply quitting Julia does not make it go away. To remove the progress bar, I need to trigger its completion myself.

```jl
active_progress = Vector{UUID}()
logger = TerminalLogger()
try
    with_logger(logger) do
        while true
            msg = deserialize(conn)
            if msg.id ∉ active_progress
                push!(active_progress, msg.id)
            end
            @info ProgressLogging.ProgressString(msg)
            if msg.done || isnothing(msg.fraction)
                idx = findfirst(x->x==msg.id, active_progress)
                deleteat!(active_progress, idx)
            end
            eof(conn) && break
        end
    end
finally
    with_logger(logger) do
        for id in active_progress
            @info ProgressLogging.Progress(; id=id, fraction=nothing, done=true)
        end
    end
    close(conn)
end
```

In this example, I save the active progress whenever a new progress message is received and remove it when they are finished. At the end of execution, for each remaining progress ID, I create a new progress message with `fraction=nothing, done=true`, which completes the progress bar with 100%.

With this, a keyboard interrupt in talker won't cause the press bars to get stuck in the listener console. Yay!

Next up, I will expand this library for use with Kerbal Space Program. Getting some real usage out of it will help shape its interfaces for the better.

If you are interested, check out the full implementation in https://github.com/RhahiSpace/RemoteLogging.jl
