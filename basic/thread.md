# Basic
## What is thread
The smallest sequence of programmed instructions that can be managed independently by a scheduler.
Implement multithreading by time slicing, the CPU switches between different software threads. The switching happens so often and rapidly that users perceive the threads running in parallel.
On a multiprocessor or multi-core system, multiple threads can execute in parallel.

## Why multiple threading is difficult
Because organize multiple tasks are difficult for us.

## Common issues of threading
1. Too many threads, without using Thread pool
2. Unless atomic, it would have problems:
3. Data races
4. Deadlocks
5. Live locks


## Race condition vs data race
http://blog.regehr.org/archives/490

By myself 
竞争条件是因为不能确定时间顺序可能造成的程序错误
数据竞争是因为访问相同的数据造成的竞争

A race condition is a flaw that occurs when the timing or ordering of events affects a program’s correctness. Generally speaking, some kind of external timing or ordering non-determinism is needed to produce a race condition; typical examples are context switches, OS signals, memory operations on a multiprocessor, and hardware interrupts.

A data race happens when there are two memory accesses in a program where both:
	• target the same location
	• are performed concurrently by two threads
	• are not reads
	• are not synchronization operations

## Simple blocking methods
These wait for another thread to finish or for a period of time to elapse. Sleep, Join, and Task.Wait are simple blocking methods.
Locking constructs	These limit the number of threads that can perform some activity or execute a section of code at a time. Exclusive locking constructs are most common — these allow just one thread in at a time, and allow competing threads to access common data without interfering with each other. The standard exclusive locking constructs are lock (Monitor.Enter/Monitor.Exit), Mutex, and SpinLock. The nonexclusive locking constructs are Semaphore, SemaphoreSlim, and the reader/writer locks.

## Signaling constructs	
These allow a thread to pause until receiving a notification from another, avoiding the need for inefficient polling. There are two commonly used signaling devices: event wait handles and Monitor’s Wait/Pulse methods. Framework 4.0 introduces the CountdownEvent and Barrier classes.


## Nonblocking synchronization constructs
These protect access to a common field by calling upon processor primitives. The CLR and C# provide the following nonblocking constructs: Thread.MemoryBarrier, Thread.VolatileRead, Thread.VolatileWrite, the volatile keyword, and the Interlocked class.
	

# A Comparison of Locking Constructs

| Construct                                                    | Purpose                                                      | Cross-process | Overhead |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------- | -------- |
| [lock](http://www.albahari.com/threading/part2.aspx#_Locking)   (Monitor.Enter / Monitor.Exit) | Ensures just one   thread can access a resource, or section of code at a time | -             | 20ns     |
| [Mutex ](http://www.albahari.com/threading/part2.aspx#_Mutex) |                                                              | Yes           | 1000ns   |
| [SemaphoreSlim](http://www.albahari.com/threading/part2.aspx#_Semaphore)   (introduced in Framework 4.0) | Ensures not more   than a specified number of concurrent threads can access a resource, or   section of code | -             | 200ns    |
| [Semaphore ](http://www.albahari.com/threading/part2.aspx#_Semaphore) |                                                              | Yes           | 1000ns   |
| [ReaderWriterLockSlim](http://www.albahari.com/threading/part4.aspx#_Reader_Writer_Locks)   (introduced in Framework 3.5) | Allows multiple   readers to coexist with a single writer    | -             | 40ns     |
| [ReaderWriterLock](http://www.albahari.com/threading/part4.aspx#_Reader_Writer_Locks)   (effectively deprecated) |                                                              | -             | 100ns    |

| Lock(this) versus   Lock(obj)                   | The disadvantage   of locking in this way is that you're not encapsulating the locking logic, so   it becomes harder to prevent [deadlocking](http://www.albahari.com/threading/part2.aspx#_Deadlocks)   and excessive [blocking](http://www.albahari.com/threading/part2.aspx#_Blocking).   A lock on a type may also seep through application domain boundaries (within   the same process). |
| ----------------------------------------------- | ------------------------------------------------------------ |
| The   misunderstanding of the object being lock | Locking doesn’t   restrict access to the synchronizing object itself in any way. In other   words, x.ToString() will not [block](http://www.albahari.com/threading/part2.aspx#_Blocking)   because another thread has called lock(x); both threads must call lock(x) in   order for blocking to occur. |
| Atomicity                                       | If a group of   variables are always read and written within the same lock, you can say the   variables are read and written atomically. Let’s suppose fields x and y are   always read and assigned within a lock on object locker:       lock (locker) { if   (x != 0) y /= x; } |
| Nested Locking                                  | A thread can   repeatedly lock the same object in a nested (reentrant) fashion:   In these   scenarios, the object is unlocked only when the outermost lock statement has   exited — or a matching number of Monitor.Exit statements have executed.   Use nested lock   instead of multiple lock to avoid dead lock       lock (locker)     lock (locker)       lock (locker)       {          // Do something...       } |
| Deadlocks                                       | A deadlock happens   when two threads each wait for a resource held by the other, so neither can   proceed.    1. lock objects in   a consistent order to avoid deadlocks   2. Relying more on   declarative and data parallelism, immutable types, and nonblocking   synchronization constructs, can lessen the need for locking. |
| Mutex                                           | A Mutex is like a   C# lock, but it can work across multiple processes. In other words, Mutex can   be computer-wide as well as application-wide. |
| semaphore                                       | A semaphore is   like a nightclub: it has a certain capacity, enforced by a bouncer. Once it’s   full, no more people can enter, and a queue builds up outside. Then, for each   person that leaves, one person enters from the head of the queue. The   constructor requires a minimum of two arguments: the number of places   currently available in the nightclub and the club’s total capacity. |
| Thread Safety                                   | A program or   method is thread-safe if it has no indeterminacy in the face of any   multithreading scenario. Thread safety is achieved primarily with locking and   by reducing the possibilities for thread interaction. |
| General-purpose   types are rarely thread-safe  | General-purpose   types are rarely thread-safe in their entirety, for the following reasons:               The development burden in full        thread safety can be significant, particularly if a type has many fields        (each field is a potential for interaction in an arbitrarily        multithreaded context).        Thread safety can entail a        performance cost (payable, in part, whether or not the type is actually        used by multiple threads).        A thread-safe type does not        necessarily make the program using it thread-safe, and often the work        involved in the latter makes the former redundant.          Thread safety is   hence usually implemented just where it needs to be, in order to handle a   specific multithreading scenario. |
| Provided thread   safe types                    | Primitive types   aside, few .NET Framework types, when instantiated, are thread-safe for   anything more than concurrent read-only access. The onus is on the developer   to superimpose thread safety, typically with exclusive locks. (The   collections in System.Collections.Concurrent are an exception.) |
| Thread safe by   default                        | *static         members are thread-safe; instance members are not.*     Read-only thread safety,         collections, for instance, are thread-safe for concurrent readers. |
| Not thread safe                                 | This means that   when writing code on the server side, you must consider thread safety if   there’s any possibility of interaction among the threads processing client   requests. |
| Thread Affinity                                 | These objects have   thread affinity, which means that only the thread that instantiates them can   subsequently access their members. Violating this causes either unpredictable   behavior, or an exception to be thrown.       On the positive   side, this means you don’t need to lock around accessing a UI object. On the   negative side, if you want to call a member on object X created on another   thread Y, you must marshal the request to thread Y. You can do this   explicitly as follows:           In WPF, call Invoke or BeginInvoke on the   element’s Dispatcher object.       In Windows Forms, call Invoke or   BeginInvoke on the control. |
| immutable object                                | An immutable   object is one whose state cannot be altered — externally or internally. The   fields in an immutable object are typically declared read-only and are fully   initialized during construction. |