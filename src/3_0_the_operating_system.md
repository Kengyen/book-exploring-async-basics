# The Operating System

The operating system stands in the centre of everything we do as programmers (well, unless you're [writing an Operating System](https://os.phil-opp.com/) or are in the [Embedded realm](https://rust-embedded.github.io/book/)),
so there is no way for us to discuss any kind of fundamentals in programming
without talking about operating systems in a bit of detail.

## Concurrency from the operating systems' perspective

<div style="color: back;  font-style: italic; font-size: 1.2em">"Operating systems has been "faking" synchronous execution since the 90s."</div>

This ties into what I talked about in the first chapter when I said that `concurrent`
needs to be talked about within a reference frame and I explained that the OS
might stop and start your process at any time.

What we call synchronous code is in most cases code that appears as synchronous to us as programmers. Neither the OS nor the CPU live in a fully synchronous world.

Operating systems use `preemptive multitasking` and as long as the operating system you're running is preemptively scheduling processes, you won't have a
guarantee that your code runs instruction by instruction without interruption.

The operating system will make sure that all important processes get some time from the CPU to make progress.

> This is not as simple when we're talking about modern machines with 4-6-8-12
> physical cores since you might actually execute code on one of the CPU's
> uninterrupted if the system is under very little load. The important part here
> is that you can't know for sure and there is no guarantee that your code will be
> left to run uninterrupted.

## Teaming up with the OS.

When programming it's often easy to forget how many moving pieces that need to
cooperate when we care about efficiency. When you make a web request, you're not
asking the CPU or the network card to do something for you, you're asking the
operating system to talk to the network card for you.

There is no way for you as a programmer to make your system optimally efficient
without playing to the strengths of the operating system. You basically don't have
access to the hardware directly.

However, this also means that to understand everything from the ground up, you'll also need to know how your operating system handles these tasks.

To be able to work with the operating system, we'll need to know how we can communicate with it and that's exactly what we're going to go through next.
