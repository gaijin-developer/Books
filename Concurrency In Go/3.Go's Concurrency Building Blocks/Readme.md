## Goroutines

A goroutine is a program that is running concurrently

Every go program has at least one goroutine, which is the main goroutine which is created and started when the process begins.

Goroutines are started by simply placing the `go` keyword befor a function

For an anonymous function, we must invoke the function immediately to use the go keyword.

```
go func (){
    //do something
}
```

Goroutines are concurrent subroutines that cannot be interrupted.

They are managed by the go runtime, and suspends them when they block and resumes them when they become unblocked

Concurrency is not a property of a coroutine(go routine), they must be hosted simultaneously by something that gives.

If there are more goroutines than green threads, the scheduler handles the distribution of the foroutines across the available threads and ensures thtat when these goroutines become blocked, other goroutines can be run.

##### The Fork-Join Model

At any point in the program a child branch can be split off and later join the parent branch to run concurrently.
Where the child joins the parent routine is called the join point, and it is at this point that a programs correctness is guaranteed and race conditions are removed.

### Go routines execute in the same address space they were created

Because the go keyword schedules a sub program to run at an undisclosed time, it is important to take note of the fact that the time at which the routine will start to run will be non-deterministic.
This means that if a goroutine is depending on a variable that may be changing values sequentially, the value of the variable may be different at the start of the routine.
EG:

```
var wg sync.WaitGroup
for _, salutation := range []string{"hello", "greetings", "good day"} {
 wg.Add(1)
 go func() {
 defer wg.Done()
 fmt.Println(salutation)
 }()
}
wg.Wait()
```

Imagine the goroutine is waiting in line, and when it gets to its turn, it goes to get the variable from the loop, but the loop moved on already and the variable is currently holding the last value in the loop, this means that that value is what the goroutine is going to get.
**This may vary depending on the system**

**_A newly minted goroutine is given a few kilobytes, which is almost always enough. When it isn’t, the run-time grows (and shrinks) the memory for storing the stack automatically, allowing many goroutines to live in a modest amount of memory. The CPU overhead averages about three cheap instructions per function call. It is practical to create hundreds of thousands of goroutines in the same address space. If goroutines were just threads, system resources would run out at a much smaller number_**

```
Estimate of the number of goroutines you can run on a system

Memory (GB)     Goroutines (#/100,000)          Order of magnitude
2^5             118.983                         6
2^6             237.967                         6
2^7             475.934                         6
2^8             951.867                         6
```

Because concurrent processes must save current state to perform context switching, we may end up using all the cpu time handling context switching between the goroutines and get little to no work done.

From the evidence provided, goroutines data communication(context switching) are about 92% faster than OS data communication (context switch).
**This depends on the machine**

If your go routines are not limited by other goroutines, you can scale your application by running more go routines.

## The Sync Package

The sync package contains the concurrency primitives that are most useful for low level memory access synchronization.

### WaitGroup

WaitGroup is a great way to wait for a set of concurrent operations to complete when
you either don’t care about the result of the concurrent operation, or you have other
means of collecting their results.
**If neither of the above statements are true, using a channel is better**
**How it works**
calls to Add increment the counter by the integer passed in, and calls to Done decrement the counter by one
Calls to Wait block until the counter is zero.

**NB: Calls to add must be done outside the goroutine**

### Mutex and RWMutex

Mutex stands for “mutual exclusion” and is a way to guard critical sections of your program.

- A critical section is an area of your program that requires exclusive access to a shared resource
- It is somewhat expensive to enter and exit a critical section, and so generally people attempt to minimize the time spent in critical sections.
- For an RWMutex, can request a lock for reading, in which case you will be granted access unless the lock is being held for writing. This means that an arbitrary number of readers can hold a reader lock so long as nothing else is holding a writer lock. It’s usually advisable to use RWMutex instead o
  Mutex when it logically makes sense

### Cond

- A rendezvous point for goroutines waiting for or announcing the occurrence
  of an event.
- An “event” is any arbitrary signal between two or more goroutines that carries no information other than the fact that it has occurred.
- Basically, with Cond, its a way for a goroutine to efficiently sleep until it is signaled to wake and check its condition.
- It looks like we’re holding this lock the entire time while we wait for the condition to occur.
- We also have a new method in this example, Signal. This is one of two methods that
  the Cond type provides for notifying goroutines blocked on a Wait call that the condi‐
  tion has been triggered.
- The other is a method called Broadcast. Internally, the runtime maintains a FIFO list of goroutines waiting to be signaled; Signal finds the goroutine that’s been waiting the longest and notifies that, whereas Broadcast sends a signal to all goroutines that are waiting.
- Broadcast is arguably the more interesting of the two methods as it provides a way to communicate with multiple goroutines at once.
- The Cond type is much more performant than utilizing channels.

### Once

- sync.Once is a type that utilizes some sync primitives internally to ensure that only one call to Do ever calls the function passed in—even on different goroutines.
- sync.Once only counts the number of times Do is called, not how many times unique functions passed into Do are called
- sync.Once as intended to guard against multiple initialization,

### Pool

- At a high level, a the pool pattern is a way to create and make available a fixed number, or pool, of things for use.
- It’s commonly used to constrain the creation of things that are expensive (e.g., database connections)
- primary interface is its Get method. When called, Get will first check whether there are any available instances within the pool to return to the caller, and if not, call its New member variable to create a new one.
- When finished, callers call Put to place the instance they were working with back in the pool for use by other processes.
- The object pool design pattern is best used either when you have concurrent processes that require objects, but dispose of them very rapidly after instantiation, or when construction of these objects could negatively impact memory.
- One thing to be wary of when determining whether or not you should utilize a Pool: if the code that utilizes the Pool requires things that are not roughly homogenous, you may spend more time converting what you’ve retrieved from the Pool than it would have taken to just instantiate it in the first place.

### Channels

- Channels are one of the synchronization primitives in Go derived from Hoare’s CSP(Communicating Sequential Processes). While they can be used to synchronize access of the memory, they are best used to communicate information between goroutines.
- nto a chan variable, and then somewhere else in your program read it off the channel. The disparate parts of your program don’t require knowledge of each other, only a reference to the same place in memory where the channel resides. This can be done by passing references of channels around your program.

**Creating a Channel**

```
var dataStream chan interface{}
dataStream = make(chan interface{})
```

**Unidirectional Channels**

```
//read only
var dataStream <-chan interface{}
dataStream := make(<-chan interface{})

//send only
var dataStream chan<- interface{}
dataStream := make(chan<- interface{})
```

- If you use a unidirectional channel as a func argument and give it a biderectional argument, Go will implicitly convert bidirectional channels to unidirectional.
- Channels are typed
- It is an error to try and write a value onto a read-only channel, and an error to read a value from a write-only channel
- Channels in Go are said to be blocking. This means that any goroutine that attempts to write to a channel that is full will wait until the channel has been emptied,
- When you receive from a channel, the second return value is a way for a read operation to indicate whether the read off the channel was a value generated by a write elsewhere in the process, or a default value generated from a closed channel.

```
intStream := make(chan int)
close(intStream)
integer, ok := <- intStream
```

- We can range over a channel and it will terminate when the channel is closed.
- Closing a channel is also one of the ways you can signal multiple goroutines simultaneously

- Buffered channels, which are channels that are given a capacity when they’re instantiated. This means that even if no reads are performed on the channel, a goroutine can still perform n writes, where n is the capacity of the buffered channel

```
var dataStream chan interface{}
dataStream = make(chan interface{}, 4)
```

- This means that we can place four things onto the channel

- An unbuffered channel is simply a buffered channel created with a capacity of 0.
- An unbuffered channel has a capacity of zero and so it’s already full before any writes. A buffered channel with no receivers and a capacity of four would be full after four writes, and block on the fifth write since it has nowhere else to place the fifth element.
- Buffered channels are an in-memory FIFO queue for concurrent processes to communicate over.
- If a buffered channel is empty and has a receiver, the buffer will be bypassed and the value will be passed directly from the sender to the.
- If a goroutine making writes to a channel has knowledge of how many writes it will make, it can be useful to create a buffered channel whose capacity is the number of writes to be made
- Be sure to ensure the channels you’re working with are always initialized first.
- Channel owners have a write-access view into the channel (chan or chan<-).
- Channel utilizers only have a read-only view into the channel (<-chan).
- Ownership as being a goroutine that instantiates, writes, and closes a channel.

### Select Statement

- This is how we’re able to compose channels together in a program to form larger abstractions
- Select statements can help safely bring channels together with concepts like cancellations, timeouts, waiting, and default values.
- Unlike switch blocks, case statements in a select block aren’t tested sequentially and execution won’t automatically fall through if none of the criteria are met.

```
var c1, c2 <-chan interface{}
var c3 chan<- interface{}
select {
case <- c1:
 // Do something
case <- c2:
 // Do something
case c3<- struct{}{}:
 // Do something
}
```

- All channel reads and writes are considered simultaneously3 to see if any of them are ready: populated or closed channels in the case of reads, and channels that are not at capacity in the case of writes. If none of the channels are ready, the entire select statement blocks. Then when one the channels is ready, that operation will proceed, and its corresponding statements will execute.
- The Go runtime will perform a pseudo random uniform selection over the set of case statements. Each has an equal chance of being selected as all the others.

**Timing out**

```
var c <-chan int
select {
case <-c:
case <-time.After(1 * time.Second):
 fmt.Println("Timed out.")
}
```

- The select statement also allows for a default clause in case you’d like to do something if all the channels you’re selecting against are blocking.

### GOMAXPROCS

This function controls the number of OS threads that will host so-called “work queues.

```
runtime.GOMAXPROCS(runtime.NumCPU())
```

- This is now automatically set to the number of logical CPUs on the host machine.
