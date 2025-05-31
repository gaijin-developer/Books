# Concurrency

This is the situation where a process that occurs simultaneously with one or more processes. This implies that both processes are making progress at the same time without one waiting on the other.
An example is humans and other animals existing and living their lives freely from each other

### Amdahl's law

This states that the performance gains from modeling a potential solution in a parallel manner is bounded by how much sequential code must be written.

## There are some common issues that makes concurrency hard

- ### Race conditions
  When two or more operations must execute in order but the code is not written in such a way that guarantees that order. This mostly results in a data race, a situation where one concurrent operation attempts to read a variable while another is attempting to write to the same variable.

E.g

```
var data int
 go func() {
 data++
 }()
 if data == 0 {
 fmt.Printf("the value is %v.\n", data)
 }
```

The function and the if conditions may both be working on the data variable.

- ## Atomicity

  Something being atomic means that its indivisible and uninterruptible.
  This means that something may be atomic in one context and not in another.
  This also means that the atomic process will happen completely without anything happening in that context simultaneously.

  if the code execution context does not expose a variable to other go routines, then the code becomes atomic

  - If something is atomic, implicitly it is safe within concurrent contexts. This can serve as a way to optimize concurrent programs.

- ## Memory Access synchronization

  In fact, there’s a name for a section of your program that needs exclusive access to a shared resource. This is called a <b>critical section</b>.
  One way to ensure the critical section has access to some data is by synchronizing the memory access by the critical sections.
  This can be achieved at a basic level with the sync package's Mutex type.

  ```
  var memoryAccess sync.Mutex
  ```

  Anytime we want to access the data variables memory, we must firs lock:

  ```
  memoryAccess.Lock()
  ```

  and unlock when we are done

  ```
  memoryAccess.Unlock()
  ```

#### Deadlock

A program is deadlocked when all concurrent processes are waiting on one another.

### According to Edgar Coffman, for a deadlock to occer, the following must be true:

- A concurrency process holds exclusive rights to a resource at any one time.
- A conn process myst simultaneously hold a resource and be waiting for an additional resource.
- a resource held by a concurrent process can only be released by that process.
- A concurrent process (P1) must be waiting on a chain of other concurrent processes (P2), which are in turn waiting on (P1).

##### We can prevent a deadlock if we can at least make one of the above statements to be untrue

## Livelock

These are programs that are actively performing concurrent operations but these operations do nothing to move the state of the program forward.

In a real world scenario, this can be likened to when two people walking towards each other keep crossing each other and no one can continue their journey because they are getting in each other's way.

Its as if the two or more concurrent processes are attempting to prevent a deadlock without coordination

## Starvation

This is when a concurrent process cannot get all the resources it needs to perform work.

This usually suggest that there is one or more greedy concurrent processes that are unfairly preventing one or more concurrent processes from accomplishing work as efficiently as possible.

When performing performance tuning, it is recommended to constrain memory access synchronization only to critical sections and broaden the scope when needed.

## Determining Concurrency Safety

When developing a new functionality, we will need to think about these few points,

- How do you expose concurrency to callers ?
- What techniques do you use to create a solution that is both easy to use and modify?
- What is the right level of concurrency for this problem ?

## Simplicity in the face of complexity

When it comes to concurrency, go does most of the heavy lifting and provides foundation for most of Go's concurrency good points.

As of Go 1.8, garbage collection pauses are generally between 10 and 100 microseconds.
Go has made it so much easier to use concurrency in your program by making you need to worry about memory.

Go's runtime also automatically handles multiplexing concurrent operations onto operating system threads. This means that concurrent operations are distributed among os threads by the go runtime.
This means when you write a go routine, go will handle the concurrent task efficiently using threads.
