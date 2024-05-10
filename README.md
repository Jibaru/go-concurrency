# Go Concurrency Concepts - Notes

This concepts was taken after reading the book Learn Concurrent Programming with Go.

## 1. Introduction

- Concurrent programming allows us to build more responsive software.
- Concurrent programs can also provide increased speedup when running on multiple processors.
- We can increase throughput even when we have only one processor if our concurrent programming makes effective use of the I/O wait times.
- Go provides us with goroutines, which are lightweight constructs for modeling concurrent executions.
- Go provides us with abstractions, such as channels, that enable concurrent executions to communicate and synchronize.
- Go allows us the choice of building our concurrent application either using the communicating sequential processes (CSP)-style model or using the classical primitives.

## 2. Dealing with Threads

- Multiprocessing operating systems and modern hardware provide concurrency through their scheduling and abstractions.
- Processes are the heavyweight way of modeling concurrency; however, they provide isolation.
- Threads are lightweight and share the same process memory space, User-level threads are even more lightweight and performant, but they require complex handling to prevent the process managing all the user-level threads from being descheduled.
- User-level threads contained in a single kernel-level thread will use only one processor at a time, even if the system has multiple processors.
- Goroutines adopt a hybrid threading system with a set of kernel-level threads containing a set of goroutines apiece. With this system, multiple processors can execute the goroutines in parallel.
- Go’s runtime uses a system of work stealing to move goroutines to other kernel level threads whenever there is a load imbalance or a descheduling takes place.
- Concurrency is about planning how to do many tasks at the same time.
- Parallelism is about performing many tasks at the same time.

### 3.Thread communication using memory sharing

- Memory sharing is one way in which multiple goroutines can communicate to accomplish a task.
- Multiprocessor and multicore systems give us hardware support and systems to share memory between threads.
- Race conditions are when unexpected results arise due to sharing resources, such as memory, between goroutines.
- A critical section is a set of instructions that should execute without interference from other concurrent executions. When interference is allowed to happen, race conditions might occur.
- Invoking the Go scheduler outside critical sections is not a solution to the problem of race conditions.
- Using proper synchronization and communication eliminates race conditions.
- Go gives us a race detector tool that helps us spot race conditions in our code.

### 4. Syncronization with mutexes

- Mutexes can be used to protect critical sections of our code from concurrent executions.
- We can protect critical sections using mutexes by calling the Lock() and UnLock() functions at the start and end of critical sections, respectively.
- Locking a mutex for too long can turn our concurrent code into sequential execution, reducing performance.
- We can test whether a mutex is already locked by calling TryLock().
- Readers–writer mutexes can provide performance improvements for read heavy applications.
- Readers–writer mutexes allow multiple readers’ goroutines to execute critical sections concurrently and provide exclusive access to a single writer goroutine.
- We can build a read-preferred readers–writer mutex with a counter and two normal mutexes.

## 5. Condition variables and semaphores

- An execution can be suspended, waiting until a condition is met, by using a condition variable together with a mutex.
- Calling Wait() on a condition variable atomically unlocks the mutex and suspends the current execution.
- Calling Signal() resumes the execution of one suspended goroutine that has called Wait().
- Calling Broadcast() resumes the execution of all suspended goroutines that have called Wait().
- If we call Signal() or Broadcast() and no goroutines are suspended on a Wait() call, the signal or broadcast is missed.
- We can use condition variables and mutexes as building blocks to build more complex concurrency tools, such as semaphores and write-preferring readers writer locks.
- Starvation occurs when an execution is blocked from a shared resource because the resource is made unavailable for a long time by other executions.
- Write-preferring readers–writer mutexes solve the problem of write starvation.
- Semaphores give us the ability to limit concurrency on a shared resource to a fixed number of concurrent executions.
- Like condition variables, semaphores can be used to send a signal to another execution.
- When used to signal, semaphores have the added advantage that the signal is stored if the execution is not yet waiting for it.

## 6. Syncronizing with waitgroups and barriers

- Waitgroups allow us to wait for a set of goroutines to finish their work.
- When using a waitgroup, a goroutine calls Done() after it finishes a task.
- To wait for all tasks to complete using a waitgroup, we call the Wait() function.
- We can use a semaphore initialized to a negative permit number to implement a fixed-size waitgroup.
- Go’s bundled waitgroup allows us to resize the group dynamically after we create the waitgroup by using the Add() function.
- We can use condition variables to implement a dynamically sized waitgroup.
- Barriers allow us to synchronize our goroutines at specific points in their executions.
- Barriers suspend the execution when a goroutine calls Wait() until all the goroutines participating in the barrier also call Wait().
- When all the goroutines participating in the barrier call Wait(), all the suspended executions on the barrier are resumed.
- Barriers can be reused multiple times.
- We can also implement barriers using condition variables.

## 7. Communication using message passing

- Message passing is another way for concurrent executions to communicate.
- Message passing is similar to our everyday way of communicating by passing a message and expecting an action or a reply.
- In Go, we can use channels to pass messages between our goroutines.
- Channels in Go are synchronous. By default, a sender will block if there is no receiver, and the receiver will also block if there is no sender.
- We can configure buffers on channels to store messages if we want to allow senders to send N messages before blocking on a receiver.
- With buffered channels, a sender can continue writing messages to the channel even without a receiver if the buffer has enough capacity. Once the buffer fills up, the sender will block.
- With buffered channels, a receiver can continue reading messages from the channel if the buffer is not empty. Once the buffer empties, the receiver will block.
- We can assign directions to channel declarations so that we can receive from or send to a channel, but not both.
- A channel can be closed by using the close() function.
- The read operation on a channel returns a flag telling us whether the channel is still open.
- We can continue to consume messages from a channel by using a for range loop until the channel is closed.
- We can use channels to collect the result of a concurrent goroutine execution.
- We can implement the channel functionality by using a queue, two semaphores, and a mutex.

# 8. Selecting channels

- When multiple channel operations are combined using the select statement, the operation that is unblocked first gets executed.
- We can have non-blocking behavior on a blocking channel by using the default case on the select statement.
- Combining a send or receive channel operation with a Timer channel on a select statement results in blocking on a channel up to the specified timeout.
- The select statement can be used not just for receiving messages but also for sending.
- Trying to send to or receive from a nil channel results in blocking the execution.
- Select cases can be disabled when we use nil channels.
- Message passing produces simpler code that is easier to understand.
- Tightly coupled code results in applications in which it is difficult to add new features.
- Code written in a loosely coupled way is easier to maintain.
- Loosely coupled software with message passing tends to be simpler and more readable than using memory sharing.
- Concurrent applications using message passing might consume more memory because each execution has its own isolated state instead of a shared one.
- Concurrent applications requiring the exchange of large chunks of data might be better off using memory sharing because copying this data for message passing may greatly degrade performance.
- Memory sharing is more suited for applications that would exchange a huge number of messages if they were to use message passing.

# 9. Programming with channels

- Communicating sequential processes (CSP) is a formal language concurrency model that uses message passing through synchronized channels.
- Executions in CSP have their own isolated state and do not share memory with other executions.
- Go borrows core ideas from CSP, with the addition that it treats channels as first-class objects, which means we can pass channels around in function calls and even on other channels.
- A quit channel pattern can be used to notify goroutines to stop their execution.
- Having a common pattern where a goroutine accepts input channels and returns outputs allows us to easily connect various stages of a pipeline.
- A fan-in pattern merges multiple input channels into one. This merged channel is closed only after all input channels are closed.
- A fan-out pattern is where multiple goroutines read from the same channel. In this case, messages on the channel are load-balanced among the goroutines.
- The fan-out pattern makes sense only when the order of the messages is not important.
- With the broadcast pattern, the contents of an input channel are replicated to multiple channels.
- In Go, having channels behave as first-class objects means that we can modify the structure of our message-passing concurrent program dynamically while the program is executing.

# 10. Concurrency patterns

- Decomposition is the process of breaking a program into different parts and figuring out which parts can be executed concurrently.
- Building dependency graphs helps us understand which tasks can be performed in parallel with others.
- Task decomposition is about breaking down a problem into the different actions needed to complete the entire job.
- Data decomposition is partitioning data in a way so that tasks on the data can be performed concurrently.
- Choosing fine granularity when breaking down programs means more parallelism at the cost of limiting scalability due to time spent on synchronization and communication.
- Choosing coarse granularity means less parallelism, but it reduces the amount of synchronization and communication required.
- Loop-level parallelism can be used to perform a list of tasks concurrently if there is no dependency on the tasks.
- In loop-level parallelism, splitting the problem into parallel and synchronized parts allows for a dependency on a previous task iteration.
- Fork/join is a concurrency pattern that can be used when we have a problem with an initial parallel part and a final step that merges the various results.
- A worker pool is useful when the concurrency needs to scale on demand.
- Pre-creating executions in a worker pool is faster than creating them on the fly for most languages.
- In Go, the performance of pre-creating a worker pool versus creating goroutines on the fly is minimal due to the lightweight nature of goroutines.
- Worker pools can be used to limit concurrency so as not to overload servers when there is an unexpected increase in demand.
- Pipelines are useful to increase throughput when each task depends on the previous one to be complete.
- Increasing the speed of the slowest node in a pipeline results in an increase in the throughput performance of the entire pipeline.
- Increasing the speed of any node in a pipeline results in a reduction in the pipeline’s latency.

# 11. Avoiding deadlocks

- A deadlock is when a program has multiple executions that block indefinitely, waiting for each other to release their respective resources.
- A resource allocation graph (RAG) shows how executions are using resources by connecting them with edges.
- In an RAG, an execution requesting a resource is represented by a directed edge from the execution to the resource.
- In an RAG, an execution holding a resource is represented by a directed edge from the resource to the execution.
- When an RAG contains a cycle, it signifies that the system is in a deadlock.
- A graph cycle detection algorithm can be used on the RAG to detect a deadlock.
- Go’s runtime provides deadlock detection, but it only detects a deadlock if all the goroutines are blocked.
- When Go’s runtime detects a deadlock, the entire program exits with an error.
- Avoiding deadlocks by using scheduling executions in a specific manner can only be done in special cases where we know beforehand which resources will be used.
- Deadlocks can be prevented programmatically by requesting resources in a predefined order.
- Deadlocks can also occur in programs that are using Go channels. A channel’s capacity can be thought of as a mutually exclusive resource.
- When using channels, take care to avoid circular waits to prevent deadlocks.
- With channels, circular waits can be avoided by sending or receiving using separate goroutines, by combining channel operations with a select statement, or by better designing programs to avoid circular message flows.

# 12. Atomics, SpinLocks and Futexes

- Atomic variables provide the ability to perform atomic updates on various data types, such as atomically incrementing an integer.
- Atomic operations cannot be interrupted by other executions.
- Applications in which multiple goroutines are updating and reading a variable at the same time can use atomic variables instead of mutexes to avoid race conditions.
- Updating an atomic variable is slower than updating a normal variable.
- Atomic variables work on only one variable at a time. If we need to protect updates to multiple variables together, we need to use mutexes or other synchronization tools.
- The CompareAndSwap() function atomically checks to see whether the value of the atomic variable has a specified value and, if it does, updates the variable with another value.
- The CompareAndSwap() function returns true only when the swap succeeds.
- A spin lock is an implementation of a mutex completely in the user space.
- Spin locks use a flag to indicate whether a resource is locked.
- If the flag is already locked by another execution, the spin lock will repeatedly try to use the CompareAndSwap() function to determine whether the flag is unlocked. Once the flag indicates that the lock is free, it can then be marked as locked again.
- When there is high contention, spin locks waste CPU cycles by looping until the lock becomes available.
- Instead of endlessly looping on the atomic variable to implement a spin lock, a futex can be used to suspend and queue the execution until the lock becomes available.
- To implement a mutex with an atomic variable and a futex, we can have the atomic variable store three states: unlocked, locked, and locked with waiting executions.
- Go’s mutexes implement a queuing system in the user space to suspend goroutines that are waiting to acquire a lock.
- The mutexes in the sync package wrap around a semaphore implementation that queues and suspends goroutines when no more permits are available.
- The mutex implementation in Go switches from normal to starvation mode in situations where newly arriving goroutines are blocking queuing goroutines from acquiring the lock.
