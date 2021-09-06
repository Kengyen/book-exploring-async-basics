# Async history

In the beginning, computers had one CPU and it executed a set of instructions written
by a programmer one by one. No scheduling, no threads, no multitasking. This was
how computers worked for a long time. We're talking back when a program
looked like a deck of these:

![Image](./images/punched_card_deck.jpg)

There were operating systems being researched even very early and when personal computing
started to grow in the 80s, operating systems like DOS were the standard on most consumer
PCs.

These operating systems usually yielded control of the entire CPU to the program currently executing and it was
up to the programmer to make things work and implement any kind of multitasking
for their program. This worked fine, but as interactive UIs using a mouse and
windowed operating systems became the norm, this model simply couldn't work anymore.

## Non-Preemptive multitasking

The first method used to be able to keep a UI interactive (and running background
processes), was accomplished by what we call `non-preemptive multitasking`.

This kind of multitasking put the responsibility of letting the OS run other tasks like responding to input from the mouse, or running a background task in the hands of the programmer.

Typically, the programmer `yielded` control to the OS.

Besides off-loading a huge responsibility to every programmer writing a program
for your platform, this was naturally error-prone. A small mistake in a programs code
could halt or crash the entire system.

> If you remember Windows 95, you also remember the times when a window hung and you could paint the entire screen with it (almost the same way as the end in Solitaire, the card game that came with Windows).
>
> This was reportedly a typical error in the code that was supposed to yield control to the operating system.

## Preemptive multitasking

While non-preemptive multitasking sounded like a good idea, it turned out to
create serious problems as well. Letting every program and programmer out there be responsible for having a responsive UI in an operating system can ultimately lead to a bad user experience since every bug out there could halt the entire system.

The solution was to place the responsibility of scheduling the CPU resources
between the programs that requested it (including to OS itself) in the hands of
the OS. The OS can stop the execution of a process, do something else, and switch back.

On a single-core machine, you can visualize this as running a program you wrote,
the OS has to stop your program to update the mouse position before it switches back to your
program to continue. This happens so frequently that we don't observe any difference whether the CPU
has a lot of work or is idle.

The OS is then responsible for scheduling tasks and does this by switching contexts on the CPU. This process can happen many times each second, not only to keep the UI responsive but it can also give some time to other background tasks and IO events.

This is now the prevailing way to design an operating system.

> If you want to learn more about this kind of threaded multitasking I recommend reading my previous book about [green threads](https://cfsamson.gitbook.io/green-threads-explained-in-200-lines-of-rust/). This is a nice introduction where you'll be able to get the basic knowledge you need about threads, contexts, stacks and scheduling.

## Hyper-threading

As CPUs evolved and added more functionality, such as several ALUs (Arithmetic Logic Units), and additional logic units, the CPU manufacturers realized that the entire CPU was never utilized fully. For example, when an operation only required some parts of the CPU, an instruction could be run on the ALU simultaneously. This became the start of hyper-threading.

You see, on your computer today that it has i.e. 6 cores, and 12 logical cores.
This is exactly where hyper-threading comes in. It "simulates" two cores on the
same core by using unused parts of the CPU to drive progress on thread "2"
simultaneously as it's running the code on thread "1". It does this by using a
number of smart tricks (like the one with the ALU).

Now, having hyper-threading, we could actually offload some work on one thread while keeping the UI
interactive by responding to events in the second thread even though we only
had one CPU core thereby utilizing our hardware better.

> You might wonder about the performance of Hyper Threading?
>
> It turns out that hyper-threading has been continuously improved since the '90s.
> Since you're not actually running two CPU's there will be some operations that
> need to wait for each other to finish. The performance gain of hyper-threading
> compared to multithreading in a single core seems to be [somewhere close
> to 30 %](https://en.wikipedia.org/wiki/Hyper-threading#Performance_claims) but
> it largely depends on the workload.

## Multicore processors

As most know, the clock frequency of processors has been flat for a long time.
Processors get faster by improving caches, branch prediction, speculative execution
and working on the processing pipelines of the processors, but the gains seems to
be diminishing.

On the other hand, new processors are so small they allow us to have many on the
same chip instead. Now, most CPUs have many cores, and most often each core will also have the ability to perform hyper-threading.

## So how synchronous is the code you write, really?

Like many things, this depends on your perspective. From the perspective of your process and the code you write for it, everything will normally happen in the order you write it.

From the OS perspective, it might, or might not, interrupt your code, pause it
and run some other code in the meantime before resuming your process.

From the perspective of the CPU, it will mostly execute instructions one at a time *.
They don't care who wrote the code though so when a hardware interrupt happens,
they will immediately stop and give control to an interrupt handler. This is how
the CPU handles concurrency.


> \* However, modern CPUs can also do a lot of things in parallel. Most CPUs are
> pipelined, meaning that the next instruction is loaded while the current is
> executing. It might have a branch predictor that tries to figure out what
> instructions to load next.
>
> The processor can also reorder instructions by using
> "out of order execution" if it believes it makes things faster this way without
> "asking" or "telling" the programmer or the OS so you might not have any guarantee that A happens before B.
>
> The CPU offloads some work to separate "coprocessors" like the FPU for floating-point calculations leaving the main CPU ready to do other tasks et cetera.
>
> As a high-level overview, it's OK to model the CPU as operating in a synchronous
> manner, but lets for now just make a mental note that this is a model with some
> caveats that become especially important when talking about parallelism,
> synchronization primitives (like mutexes and atomics) and the security of computers
> and operating systems.
