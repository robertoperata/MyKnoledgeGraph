---
tags:
  - threads
  - java
feature:
type:
author: Murice Naftalin
source:
---
# Java Threads Demystified
https://github.com/MauriceNaftalin/demystifying-threads

## Introduction to Java Concurrency
Concurrency: multiple tasks running in overlapping time periods
Processes: run applications "simultaneously" even on single-CPU machines
Responsive User Interfaces: don't freeze for long running task
Design Convenience: 
Resource Utilisation: 
### Processes vs Threads
Processes:
- insulated from each other; communicate via files, signals, sockets, pipes, etc
- no shared memory (by default)
Threads (lightweight processes):
- run within a single process, share process resources
- in particular share memory
#### Benefits of Threads
- gain concurrency benefits within single process
- resource utilisation, responsive interfaces, design convenience
- simplified handling of asynchronous events
- divide work of a single process over multiple CPUs
#### Drawbacks of Threads
- safety
	- inadequately synchronised code can yield incorrect results (race condition)
- liveness
	- acquiring locks in the wrong order can deadlock your program
- Performance
	- overhead of thread creation and switching

Race Condition: when there are two different signals racing to reach the same component and which one of them arrives first will control what happens to the result.

>[!question] what does `.join` do in Threads?

Concurrency:
- multiple task running in overlapping periods
Parallelism:
- concurrent tasks (typically the same task) running on different data at the same time

#### Livelock

#### Starvation
### Threads and Synchronisation
Platform threads are wrapper around native operating system threads
In Java a Thread is like a "one shot computer" which you give some work to do.
```java
Runnable r = new Runnable() {
	public void run() {
		// do something
	}
}
```
or
```java
Runnable r = (() -> {
 // do something
})
```

The Runnable is **supplied** to the thread at construction time:
```java
var t = new Thread(r);
```

and the computer is then started:
```java
t.start();
```
or
```java
var thread3 = Thread.ofPlatform().start(r);
```

The thread dies when the `run` method exits.


#### Pitfalls of Multithread Programming
Multiple threads running on modern hardware can produce unexpected results:
- caches may duplicate values
- instructions may execute out of order
- 64-bit numbers may be processed in two parts by different threads (word tearing)

Probably most common are **race conditions**

A race condition occurs when the behavior of a system depends unpredictably on the timing of uncontrollable events

The Java memory model tells you what you need to do to avoid these pitfalls
most important rule is about **synchronisation**:
- The JMM guarantees that everything done by a thread  before it *releases* a monitor lock is visible to another thread after it has *acquired* the monitor lock
A monitor is a piece of operator that contains a lock which a thread con acquire be entering a critical section. Every java object is a monitor.
```java
synchronized(this) {
	counter++;
}
```

before it can execute that (counter++) which is the body of the critical section (marked by synchronized) it must acquire the monitor lock.

Code in a critical section can be executed only by one thread at a time.
So synchronisation has two important effects:
- it prevents threads from interfering with one another in a critical section
- it ensures that changes made by one thread are visible to another - provided that those changes are made by the first thread before it releases a monitor which the second thread subsequently acquires.

`volatile` does provides the visibility guarantee but without mutual exclusion

the platform now offers other ways to synchronise:
- explicit locks `java.util.concurrent.locks.Lock`
- Atomic variables
## Writing Thread-safe Code
- Identify the fields that form the object's state
- Identify the invariants that constrain the state fields
- Establish a policy for managing concurrent access to the state and **document it** using `@ThreadSafe` and `@GuardedBy`
### Confinement
When an object can only ever be accessed by one thread at a time, say it is ***confined***
3 ways to confine an object to a thread:
- instance confinement
	- depends on encapsulation
- Thread confinement
	- no need for synchronisation
	- ThreadLocal
- Stack confinement
#### Immutability
Immutable object are always thread-safe
an immutable object is one whose state graph can't observably change after construction
java object can be immutable. One way is to define their class such that itt
- has no mutators (setter methods, methods that change the state)
- has only `private final` fields
- has exclusive access to any mutable components
***Immutability is the simplest way to achieve thread safety***

#### Encapsulation (instance confinement)
Encapsulation prevents state of an object from being exposed externaly
- no public fields, only getters
- also all the chain of object related. If an object holds a map of some object, the single fields of those object are part of the state
	- un-mutability do not fully protect to state change. in fact prevents to modify the object itself but not the content
```java
private Map<Vehicle, MutablePoint> vehicleLocations;

public Map<Vehicle, MutablePoint> getVehicleLocations() {
	return Collections.unmodifiableMap(vehicleLocations);
}

// client
vehicleTracker.getVehicleLocations().get(vehicles.get(0)).setX(100); //PROBLEM

```
- defensive copy (shallow) has the same problem because the internal object are still referenced
```java
public Map<Vehicle, MutablePoint> getVehicleLocations() {
	return new HashMap<Vehicle, MutablePoint>(vehicleLocations);
}
//PROBLEM
vehicleTracker.getVehicleLocations().get(vehicles.get(0)).setX(100); 

```

- defensive copy (deep) clone also the internal
```java
private static Map<Vehicle, MutablePoint> defensiveCopy(Map<Vehicle, MutablePoint> vehicleLocations) {  
    final HashMap<Vehicle, MutablePoint> copy = new HashMap<>(vehicleLocations);  
    for (Vehicle v : vehicleLocations.keySet()) {  
        copy.put(v, new MutablePoint(copy.get(v)));  
    }  
    return copy;  
}
```
## Task and Task Execution
### Per-thread vs Pooling
Starting a new thread has a significant overhead (~ 1MB, 1ms)
- for small tasks the overhead may predominate
- too many requests being concurrently handled => too many threads!
For resources that are expensive to create, pooling is the best answer
- can limit the number of requests begin handled by queuing them up
- pooled threads can be reused indefinitely
### Task and Task execution
A task is a unit of work
ideally tasks should be:
- independent of the state of other tasks
- fine-grained (relatively small fraction of application's computation requirement)
for example single client request to a server application
in designing a concurrent system think about task not threads!
>[!todo] Look at interface Executor 

- `Executors.newFixedThreadPool()` keeps a permanent total number of threads
- `Executors.newCachedThreadPool()` pool size varies with demand
- `Executors.newSingleThreadExecutor()` replace the single thread if it dies unexpectedly

### Callable
`Runnerable`can't return a result. What if we want to collect together the results of concurrently executing tasks?
 Result-bearing version of `Runnable` is `Callable<v>`
### Future
You may want to interact with a task after it's been submitted for execution.
is it completed? Has id been cancelled? You want to cancel it!
Get the result (waiting is necessary)

`Future<V>` allows to observe and manage tasks after submission.

`Future` are created by `ExecutorService` and `CompletionService`
- you submit a `Callable<V>` to get back a `Future<V>`
- you submit a `Runnable` to get a `Future<?>`
### CompletionService
Allows to submit `Callables`and get them back in order of completion

### The Fork/join Framework
Fork-Join framework implements recursive decomposition
- repeatedly dividing a large task into smaller parts
- processing each small part independently
- joining the results together
To use the Fork/Join framework
- define a `RecursiveAction` (for void tasks) or a `RecursiveTask<v>` (for result-bearing tasks)
- template for `RecursiveAction`: to override `compute`
```
procted void compute() {
	if (workload.length() > THERSHOLD) {
		ForkJoinTask.invokeAll(createSubtasks());
	} else {
		process(workload);
	}
}
```
- submit your `RecursiveAction` or `RecursiveTask<V>` to the common Fork/Join pool, or one that you have created.
### Parallel Streams
Much easier way to get Fork/Join functionality is to use parallel streams
only need to insert a call to `parallel()`
but conditions have to be right
- the workload should be computationally intensive
	- and per element independent
- the total time to execute the sequential version should be around 50 microseconds or greater 
- the stream source should be efficiently splittable (ArrayList yes, LinkedList no)
- there should be enough cores to outweigh the overload
### CompletableFuture
A `Future` with additional logic
Flexible way to model complex concurrent workflows
`CompletableFuture.supplyAsync(Supplier<U>)`
- Factory method: creates a `CompletableFuture` that completes with the value obtained by calling the `Supplier` asynchronously
`CompletableFuture<Void> thenAccept(Consumer<T>)`
- instance method: crates a new `CompletableFuture` that, when this future completes, executes with tis result
likely to be less used with the advent of virtual threads
## Concurrent utilities
### Storage Collection
- List, Set, Map
Original Collections Framework implementation not thread-safe
ArrayList, HashSet, HashMap 
"Why pay for thread safety if you don't need it?"
- can use with *client-side locking*
- or with synchronized wrappers
- Look out for `ConcurrentmodificationException`
>[!tip] Look ArrayListStack has the underlying synchornized doesn't work properly

>[!hint] In general `Collections.synchonizedList()` doesn't work
### Concurrent Storage Collection
Java 5 introduced concurrent storage collection
most imporatn is `ConcurrentHashMap`
- support concurrent read operations
- thread-safe and atomic operations
- can iterate over the collection while accessing it
	- result not deterministic
- client-side locking not possible
- but also not necessary because of these atomic actions:
	- `V putIfAbsent` atomic (doesn't test and compute )
	- `boolean remove`
	- `boolean replace`
	- `V replace`
	- `V computeIfPresent`
	- `V compute`
	- `V merge`
Event a threadSafe collection could have race conditions problems:
```java
private static Map<String, Integer> map = new ConcurrentHashMap<>();
public static void main(String[] args) throws InterruptedExecption {
	Thread t1 = Thread.ofPlatform().start()(() -> {
		for (int i = 0; 1 < 1000; i++) {
			map.put("key1", map.getOrDefault("key1", 0) + 1);
		}
	});
	Thread t2 = Thread.ofPlatform().start()(() -> {
		for (int i = 0; 1 < 1000; i++) {
			map.put("key1", map.getOrDefault("key1", 0) + 1);
		}
	});
}
```

 `getOrDefault` the compound operation (get + compute + put) is not atomic. Each individual operation is  thread-safe, but the read-modify-write sequence as a whole is not. 
to fix we must replace with:
```java
map.merge("key1", 1, Integer::sum);
```

**Nothing similar for `List`**

For `Set` use `ConcurrentHashMap.newKeySet()`
For `List` and `Set`: `CopyOnWriteArrayXxx`
- useful for supporting many concurrent read operations not useful if you have many concurrent writes
### Queues
Blocking queues
- designed for workflow scenarios
- producer-consumer pattern
- queues can be bounded or unbounded
blocking queues force consumer to wait if there are no tasks available
bounded blocking queues force producers to wait if the queue is full
workflow can be throttled, it can be evened out because the queue acts as a buffer which prevents producers from producing too much and allows consumers to wait until there's task available.
>[!tip] Should we avoid client locking whenever possible? Yes
>it's very difficult to mantain and is unreliable.
### Synchronizers
Utilities that coordinate control flow of threads
- Semaphore
	- manages a finite set of permits
	- threads wanting to access a resource must wait for a permit
- CountDownLatch - gate that can block threads
	- closed when created
	- maintains a count of unavailable resources
	- when count has reached zero, gate opens
- CyclicBarrier, Phaser, Exchanger, SynchronousQueue
## Virtual Threads
The reactive programming model has been very successful
it handles the problem of IO blocking
	the cost is a very difficult programming model
Virtual threads are a different approach
	same API as Thread but very lightweight (0.001x)
	feasible to create millions of virtual threads
very low cost of blocking
implementation uses small number of platform threads
called "carrier threads"
virtual threads scheduled differently from platform threads
	0s schedulers are pre-emptive
		allow each thread a time-slice then do a context switch
	Virtual threads are scheduled non-preemptively
		car run as long as they want - or until they block
so only useful for blocking code
do not use virtualThread for CPU intense, but use for IO intense
## Structured concurrency
A `StructuredTaskScope` defines subtask, executed asynchronously
	like the Fork/join framework but subtasks can be heterogenous
	like `Completablefuture` but with a much easier programming model

```java
Callable<String> task1 = ...
Callable<Integer> task2 = ...

try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
	Subtask<String> subtask1 = scope.fork(task1);
	Subtask<Integer> subtask2 = scope.fork(task2);
	scope.join();
	scope.throwIfFailed();
	result = combine(subtask1.get, subtask2.get);
}
```


Look further:
- O'Reilly on-demand courses:
	- java Mulththreading and Parallel Programming Masterclass (Cosmin)
	- Java concurrency 2nd edition (Douglas Schmidt)
- Java Memory Model - JSR 133
- Java Concurrency in Practice (Brian Goetz)
- Java Generics and Collections
- Masterling Lambdas (Maurice Naftalin) - parallel stream perfomrance