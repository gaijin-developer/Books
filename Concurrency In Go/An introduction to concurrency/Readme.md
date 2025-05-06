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
