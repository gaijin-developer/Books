When working with concurrent code, there are some options for safe operation including using sync.Mutex and also Channels.
Other ways that concurrent code can be safely executed include using Immutable data or data protected by confinement.

- For an immutable data, each concurrent process must create their own copy of the data with the desired changes. This can make programs faster if it leads to smaller critical sections.
- In Go, you can work with copies of values by not using the pointers of the data.

#### Confinement

The idea of ensuring information is only ever available from one concurrent process.

We can use either use a read only channel or write only channel to limit the actions of a go routine to either write only or read only.

If the data can be broken up for different parts of it to be worked on, it becomes impossible do do the wrong thing by accessing a part of the data another part is already working on.
EG.

```
go printData(&wg, data[:3])
go printData(&wg, data[3:])

```

Data synchronization comes with a cost and if possible, avoid using it.

#### For-Select Loop

```
for {
    select {
        case <-done:
           return
        default:
    }
}
```

In the example above, if the done channel isnt closed, the code will continue to run other code in the body of the for loop. The default part of the select statement will be executed.

If the done channel gets closed, the execution will exit the loop.

#### Preventing Go routine leaks

Important.
**Go routines are multiplexed by the runtime onto the OS threads thereby taking away the need to worry about that level of abstraction**
**Goroutines are not garbage collected by the runtime so even though their memory footprint is small its important to make sure they don't leak**

##### Paths to termination for a go Routine

- work done
- when an unrecoverable error happens
- when its terminated by the developer.

We can terminate a go routine using a signal to inform children go routines to stop working. This is usually a read-only channel named "done" by convention and is passed from the parent goroutine to the child.

**By Convention this done channel is usually passed as the first argument**

If a goroutine is responsible for creating a goroutine, it is also responsible for ensuring it can stop the goroutine.

#### The Or-Channel

A lot of channels can be combined to one function (called or) and this function will return one channel that closed if any of the channels it takes in close.
This is achieved using recursion and goroutines.

, or if you just prefer a one-liner, you can combine these
channels together using the or-channel pattern. This
This pattern creates a composite done channel through recursion and goroutines.

<pre>
```
var or func(channels ...<-chan interface{}) <-chan interface{}
or = func(channels ...<-chan interface{}) <-chan interface{} {
 switch len(channels) {
 case 0:
return nil
 case 1:
 return channels[0]
 }
 orDone := make(chan interface{})
 go func() {
 defer close(orDone)
 switch len(channels) {
 case 2:
 select {
 case <-channels[0]:
 case <-channels[1]:
 }
 default:
 select {
 case <-channels[0]:
 case <-channels[1]:
 case <-channels[2]:
 case <-or(append(channels[3:], orDone)...):
 }
 }
 }()
 return orDone
}
```
</pre>

### Error Handling

Errors are usually pushed back up the stack when they happen but at some point, something needs to be done with the error.

The concurrent process should send their errors to another part of your program that has complete information about the state of your program and can make more informed decision about what to do with the error.

The handler of the error should be the one with more context about the running program and can make more intelligent decisions about what to do with errors.

If the goroutine can produce error then those error should be coupled with your result type.

### Pipelines

A pipeline is a powerful tool to use when your program needs to process streams or batches of data.
In this scenario a series of stages where data is taken in , worked on and pass the data back out to the next stage.

Stages are modified independent of one another. Each stage can be processed concurrent to downstream or upstream stages and you can fan out or rate-limit portions of the pipeline.

Conditions of a pipeline stage:

- A stage consumes and returns the same type.
- A stage must be reified by the language so that it may be passed around. Functions in Go are reified and fit this purpose nicely.

If the pipeline/stage takes in chunks of data and push out the same, it is said to be performing batch processing. This just means that they operate on chunks of data all at once instead of one discrete value at a time.

#### Best practices for constructing pipelines

Channels are great for pipelines because they can receive and emit values and they can safely be used concurrently as well as ranged over.

A for-select pattern can be used to await data on a done channel. Such functions are called generators and are made interruptible using the done channel. This is because the channel we are ranging over will be closed when preempted and therefore our range will break when this occurs.

If a stage is blocked on sending a value, it is preemtable thanks to the select statement.

### Fan-Out / Fan-In

This method helps us to reuse stages of a pipeline to parallelize pulls from an upstream page.

Fan-out is a term to describe the process of starting multiple goroutines to handle input from the pipeline, and fan-in is a term to describe the process of combining multiple results into one channel.

### The or-done channel

When working with unrelated channels, you can't assume they'll close when your goroutine is canceled, so you should use a select with the done channel to safely handle cancellation.

### The Tee channel

Sometimes you may want to split values coming in from a channel so that you can send them off into two separate areas of your codebase

You can pass it a channel to read from, and it will return two separate channels that will get the same value.

<pre>
```
tee := func(
 done <-chan interface{},
 in <-chan interface{},
) (_, _ <-chan interface{}) { <-chan interface{}) {
 out1 := make(chan interface{})
 out2 := make(chan interface{})
 go func() {defer close(out1)
 defer close(out2)
 for val := range orDone(done, in) {
 var out1, out2 = out1, out2
 for i := 0; i < 2; i++ {
 select {
 case <-done:
 case out1<-val:
 out1 = nil
 case out2<-val:
 out2 = nil
 }
 }
 }
 }()
 return out1, out2
}

```
</pre>

### The bridge channel

If we instead define a function that can destructure the channel of channels into a simple channel—a technique called bridging the channels.

### Queuing

Sometimes it’s useful to begin accepting work for your pipeline even though the pipeline is not yet ready for more. This process is called queuing.
Once your stage has completed some work, it stores it in a temporary location in memory so that other stages can retrieve it later and your stage doesn’t need to hold a reference to it.

One example of the first situation is a stage that buffers input in something faster (memory) than it is designed to send to (e.g., disk). This is, of course, the entire purpose of Go’s bufio package.

The buffered write is faster than the unbuffered write. This is because in bufio.Writer, the writes are queued internally into a buffer until a sufficient chunk has been accumulated, and then the chunk is written out Chunking is faster because bytes.Buffer must grow its allocated memory to accommodate the bytes it must store.

Usually anytime performing an operation requires an overhead, chunking may increase system performance.
Some examples of this are opening database transactions and allocating contiguous space.

Queuing should be implemented either:

- At the entrance to your pipeline.
- In stages where batching will lead to higher efficiency.

**In queuing theory, there is a law that—with enough sampling—predicts the throughput of your pipeline. It’s called Little’s Law,- : L=λW**

A stable system is one in which the rate that work enters the pipeline, or ingress, is equal to the rate in which it exits the system, or egress.

If we want to decrease W, the average time a unit spends in the system by a factor of n, we only have one option: to decrease the average number of units in the system: L/n = λ \* W/n.

### The context Package

It helps us to preempt operations because of timeouts, cancellation, or failure of another portion of the system.

This is the type that will flow through your system much like a done channel does.
Each function that is downstream from your top-level concurrent call would take in a Context as its first argument Deadline returns ok==false when no deadline is set.

Done may return nil if this context can never be canceled.
Err returns a non-nil error value after Done is closed.
Value returns the value associated with this context for key or nil if no value is associated with key.

There’s a Done method which returns a channel that’s closed when our function is to be preempted.
There’s also some new, but easy to understand methods: a Deadline function to indicate if a goroutine will be canceled after a certain time, and an Err method that will return non-nil if the goroutine was canceled

Request-specific information needs to be passed along in addition to information about preemption.

Any blocking operations within a goroutine need to be preemptable so that it may be canceled.

The Context type will be the first argument to your function. The functions all generate new instances of a Context with the options relative to these functions.

**WithCancel** returns a new Context that closes its done channel when the returned cancel function is called.
**WithDeadline** returns a new Context that closes its done channel when the machine’s clock advances past the given deadline.
**WithTimeout** returns a new Context that closes its done channel after the given timeout duration.

If the function doesn’t need to modify the cancellation behavior, the function simply passes on the Context it was given.

For this reason, it’s important to always pass instances of Context into your functions.

Background (context.Background()) simply returns an empty Context. TODO is not meant for use in production, but also returns an empty Context

TODO’s (context.TODO()) intended purpose is to serve as a placeholder for when you don’t know which Context to use.

**context.WithTimeout**, will automatically cancel the returned Context after 1 second, thereby canceling any children it passes the Context into, namely locale.

**context.WithValue**
The other half of what the context package provides: a data-bag for a Context to store and retrieve request-scoped data.

The only qualifications for a value to be used in a context value are that:

- The key you use must satisfy Go’s notion of comparability; that is, the equality operators == and != need to return correct results when used.
- Values returned must be safe to access from multiple goroutines.
