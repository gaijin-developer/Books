# Concurrency

Concurrency is a property of the running program.

In a program we perceive to be running in parallel(on a single core system), they are really running in a sequential manner faster than it is distinguishable. The CPU context switches to share time between different programs.
This over some time makes the program appear to be running in parallel.

❗️However if we were to run the same binary on a machine with two cores, the program might actually run in parallel.

We dont write parallel code, we only write concurrent code.
Parallelism is a property of our runtime, not the code.

Parallelism depends on time or context.

#### A context is the process in which our program runs, its operating system thread or its machine.

In Go, **channels** are used to model a lot of things concurrently when the machine couldn't handle that many threads.

**CSP - Communicating Sequential Processes was published by Antony Hoare**

Hoare’s CSP programming language contained primitives to model input and output, or communication, between processes correctly.
Hoare applied the term processes to any encapsulated portion of logic that required input to run and produced output other processes would consume.

**A guarded command is simply a statement**
with a left- and righthand side, split by a →. The lefthand side served as a conditional or guard for the righthand side in that if the lefthand side was false or, in the case of a command, returned false or had exited, the righthand side would never be executed.
Combining these with Hoare’s I/O commands laid the foundation for Hoare’s communicating processes, and thus Go’s channels.

Most popular languages favor sharing and synchronizing access to the memory to CSP’s message-passing style. Go is one of the first languages to incorporate principles from CSP in its core, and bring this style of concurrent programming to the masses.

If we were to draw a comparison between concepts in the two ways of abstracting concurrent code, we’d probably compare the goroutine to a thread, and a channel to a
mutex.

How we naturally think about the problem maps directly to the natural way to code things in Go.
Go’s runtime multiplexes goroutines onto OS threads
automatically and manages their scheduling for us. Because Go’s runtime is managing the scheduling of goroutines for you, it can introspect on things like goroutines blocked waiting for I/O and intelligently reallocate OS threads to goroutines that are not blocked.

Channels, for instance, are inherently composable with other channels. This makes
writing large systems simpler because you can coordinate the input from multiple
subsystems by easily composing the output together.

The **select** statement is the complement to Go’s channels and is what enables all the difficult bits of composing channels and is what enables all the difficult bits of composing channels. Select statements allow you to efficiently wait for events, select a message from competing channels in a uniform random way, continue on if there are no messages waiting etc.

# Go’s Philosophy on Concurrency

In particular, consider structuring your program so that only one goroutine at a time is ever responsible for a particular piece of data.

### One of Go’s mottos is “Share memory by communicating, don’t communicate by sharing memory.”

### Decision points to help to choose a concurrency pattern.

- **Are you trying to transfer ownership of data? Channels**
  - If you have a bit of code that produces a result and wants to share that result with
    another bit of code, what you’re really doing is transferring ownership of that
    data.
  - Channels help us communicate this concept by encoding that intent into the channel’s type
  - One large benefit of doing so is you can create buffered channels to implement a cheap in-memory queue and thus decouple your producer from your consumer.
- **Are you trying to guard internal state of a struct?**

  - This is a great candidate for memory access synchronization primitives, and a
    pretty strong indicator that you shouldn’t use channels. This way, you can hide the implementation detail of locking your critical section from your callers.

  - Are you trying to coordinate multiple pieces of logic?
  - Channels are inherently more composable than memory access synchronization primitives. Having channels everywhere is expected and encouraged!
  - If you find yourself struggling to understand how your concurrent code works, why a deadlock or race is occurring, and you’re using primitives, this is probably a good indicator that you should switch to channels.

- **Is it a performance-critical section?**
  - Using memory access synchronization primitives may help this critical section perform under load. This is because channels use memory access synchronization to operate.

Stick to modeling your problem space with goroutines, use them to represent the concurrent parts of
your workflow, and don’t be afraid to be liberal when starting them.

Go’s philosophy on concurrency can be summed up like this: aim for simplicity, use channels when possible, and treat goroutines like a free resource.
