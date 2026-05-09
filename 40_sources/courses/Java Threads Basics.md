---
tags:
  - java
  - threads
  - concurrency
feature: 
type: course
author: 
source: 
---
# Java Threads Basics

## 1. Foundations

### 1.1 Concurrency vs Parallelism
Concurrency: multiple tasks running in overlapping time periods
Parallelism: concurrent tasks running on different data at the same time

### 1.2 Processes vs Threads
#### Processes
- Insulated from each other
- Communicate via files, signals, sockets, pipes
- No shared memory (by default)

#### Threads (Lightweight Processes)
- Run within a single process
- Share process resources
- Share memory

### 1.3 Benefits of Concurrency
- Resource utilisation
- Responsive user interfaces
- Design convenience
- Simplified handling of asynchronous events
- Divide work over multiple CPUs

### 1.4 Risks of Concurrency
#### Safety (Race Conditions)
A race condition occurs when the correctness of a computation depends on the relative timing or interleaving of multiple threads.

#### Liveness (Deadlock)
Acquiring locks in the wrong order can deadlock your program. A deadlock occurs when two or more threads are blocked forever, each waiting for the other.

#### Livelock
Threads are not blocked but cannot make progress because they keep responding to each other's actions.

#### Starvation
A thread is perpetually denied access to resources it needs because other threads are monopolising them.

#### Performance
Overhead of thread creation, context switching, and synchronisation.

---

## 2. Thread Fundamentals
Platform threads are wrapper around native operating system threads
In Java a Thread is like a "one shot computer" which you give some work to do.
### 2.1 Creating Threads
#### Using Runnable
```java
Runnable r = () -> {
    // do something
};
var t = new Thread(r);
t.start();
```

#### Extending Thread Class
```java
class MyThread extends Thread {
    @Override
    public void run() {
        // do something
    }
}
new MyThread().start();
```

#### Using Thread.ofPlatform() (Java 21+)
```java
var thread = Thread.ofPlatform().start(() -> {
    // do something
});
```

### 2.2 Thread Lifecycle
#### Thread States
- **NEW**: Thread created but not yet started
- **RUNNABLE**: Thread executing or ready to execute
- **BLOCKED**: Thread waiting to acquire a monitor lock
- **WAITING**: Thread waiting indefinitely for another thread
- **TIMED_WAITING**: Thread waiting for a specified time
- **TERMINATED**: Thread has completed execution

#### State Transitions
- NEW -> RUNNABLE: `start()` called
- RUNNABLE -> BLOCKED: waiting for monitor lock
- RUNNABLE -> WAITING: `wait()`, `join()`, `park()`
- RUNNABLE -> TIMED_WAITING: `sleep()`, `wait(timeout)`, `join(timeout)`
- RUNNABLE -> TERMINATED: `run()` exits

### 2.3 Thread Control Methods
#### start() vs run()
- `start()`: creates new thread and invokes `run()` in that thread
- `run()`: executes in the current thread (no new thread created)

#### join()
Waits for a thread to complete before continuing.
```java
Thread t = new Thread(task);
t.start();
t.join(); // waits for t to finish
```

#### sleep()
Pauses the current thread for a specified duration.
```java
Thread.sleep(1000); // sleep for 1 second
```

#### yield()
Hints to the scheduler that the current thread is willing to yield its current use of a processor.

#### interrupt() and InterruptedException
Mechanism for cooperative thread cancellation.
```java
thread.interrupt(); // sets interrupt flag

// In the target thread:
if (Thread.interrupted()) {
    // handle interruption
}
```

### 2.4 Thread Properties
#### Priority
```java
thread.setPriority(Thread.MAX_PRIORITY); // 10
thread.setPriority(Thread.NORM_PRIORITY); // 5
thread.setPriority(Thread.MIN_PRIORITY); // 1
```

#### Daemon Threads
Background threads that don't prevent JVM shutdown.
```java
thread.setDaemon(true); // must be set before start()
```

#### Thread Naming
```java
Thread t = new Thread(task, "MyThreadName");
// or
thread.setName("MyThreadName");
```

#### ThreadGroup
Logical grouping of threads (largely obsolete, prefer Executors).

### 2.5 Exception Handling
#### UncaughtExceptionHandler
```java
thread.setUncaughtExceptionHandler((t, e) -> {
    System.err.println("Thread " + t.getName() + " threw: " + e);
});
```

---

## 3. Java Memory Model

### 3.1 Hardware Realities
#### CPU Caches
Each CPU core has its own cache; changes may not be immediately visible to other cores.

#### Instruction Reordering
Compiler and CPU may reorder instructions for optimisation.

#### Word Tearing
64-bit values (long, double) may be processed in two 32-bit operations (for non-volatile variables).

### 3.2 Happens-Before Relationship
Defines when memory writes by one thread are guaranteed to be visible to reads by another thread.

#### Program Order Rule
Each action in a thread happens-before every subsequent action in that thread.

#### Monitor Lock Rule
An unlock on a monitor happens-before every subsequent lock on that same monitor.

#### Volatile Variable Rule
A write to a volatile field happens-before every subsequent read of that field.

#### Thread Start Rule
A call to `Thread.start()` happens-before any action in the started thread.

#### Thread Join Rule
All actions in a thread happen-before any other thread successfully returns from `join()` on that thread.

#### Transitivity
If A happens-before B, and B happens-before C, then A happens-before C.

### 3.3 Visibility
Changes made by one thread may not be visible to other threads without proper synchronisation.

### 3.4 Atomicity
An operation is atomic if it appears to happen all at once (indivisible).
- Reading/writing references is atomic
- Reading/writing primitives (except long/double) is atomic
- Compound operations (i++, check-then-act) are NOT atomic

---

## 4. Synchronisation Primitives

### 4.1 Intrinsic Locks (synchronized)
#### Synchronized Blocks
```java
synchronized(lockObject) {
    // critical section
}
```

#### Synchronized Methods
```java
public synchronized void method() {
    // locks on 'this'
}

public static synchronized void staticMethod() {
    // locks on Class object
}
```

#### Reentrancy
A thread can acquire a lock it already holds. Intrinsic locks are reentrant.

### 4.2 wait/notify/notifyAll
#### Object Monitor Methods
Must be called while holding the object's lock.

```java
synchronized(lock) {
    while (!condition) {
        lock.wait(); // releases lock and waits
    }
    // proceed
}

synchronized(lock) {
    condition = true;
    lock.notifyAll(); // wakes all waiting threads
}
```

#### Guarded Blocks Pattern
Always use `while` loop (not `if`) due to spurious wakeups.

#### Spurious Wakeups
Threads may wake up without being notified; always recheck condition.

### 4.3 volatile
Provides visibility guarantee without mutual exclusion.
```java
private volatile boolean flag = false;
```
- Writes are immediately visible to other threads
- Prevents instruction reordering around volatile access
- Does NOT provide atomicity for compound operations

### 4.4 Explicit Locks (java.util.concurrent.locks)
#### Lock Interface
```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```

#### ReentrantLock
- Supports fairness policy
- Supports interruptible lock acquisition
- Supports try-lock with timeout

```java
Lock lock = new ReentrantLock(true); // fair lock
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        // critical section
    } finally {
        lock.unlock();
    }
}
```

#### ReadWriteLock / ReentrantReadWriteLock
Allows multiple concurrent readers OR single writer.
```java
ReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock = rwLock.readLock();
Lock writeLock = rwLock.writeLock();
```

#### StampedLock
Optimistic reading for better performance in read-heavy scenarios.

#### Condition Interface
More flexible alternative to wait/notify.
```java
Condition condition = lock.newCondition();
condition.await();   // like wait()
condition.signal();  // like notify()
condition.signalAll(); // like notifyAll()
```

#### Lock vs synchronized Comparison
| Feature | synchronized | Lock |
|---------|--------------|------|
| Automatic release | Yes | No (finally block required) |
| Interruptible | No | Yes |
| Try-lock | No | Yes |
| Fairness | No | Optional |
| Multiple conditions | No | Yes |

### 4.5 Atomic Variables
#### AtomicInteger, AtomicLong, AtomicBoolean
```java
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();
counter.compareAndSet(expected, newValue);
```

#### AtomicReference
```java
AtomicReference<User> userRef = new AtomicReference<>(initialUser);
userRef.compareAndSet(oldUser, newUser);
```

#### Compare-And-Swap (CAS) Operations
Lock-free atomic operation: update value only if current value matches expected value.

#### AtomicFieldUpdaters
Atomic operations on volatile fields without wrapper objects.

#### LongAdder, LongAccumulator
Higher throughput than AtomicLong under high contention.
```java
LongAdder adder = new LongAdder();
adder.increment();
long sum = adder.sum();
```

---

## 5. Thread Safety Strategies

### 5.1 Immutability
Immutable objects are always thread-safe.

Requirements for immutability:
- No mutator methods (setters)
- All fields are `private final`
- Exclusive access to mutable components
- Class is final (or ensure no subclass breaks immutability)

### 5.2 Confinement
#### Stack Confinement
Object is only accessible via local variables.

#### Thread Confinement
Object is only ever accessed by one thread.

#### ThreadLocal
Per-thread storage.
```java
ThreadLocal<DateFormat> dateFormat = ThreadLocal.withInitial(
    () -> new SimpleDateFormat("yyyy-MM-dd")
);
DateFormat df = dateFormat.get();
```

#### Instance Confinement
Object is encapsulated within another object with proper locking ( depends on encapsulation). 

### 5.3 Encapsulation
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
#### Defensive Copying
Return copies instead of internal mutable objects.

**Shallow copy (may be insufficient):**
defensive copy (shallow) has the same problem because the internal object are still referenced
```java
public Map<Vehicle, MutablePoint> getVehicleLocations() {
	return new HashMap<Vehicle, MutablePoint>(vehicleLocations);
}
//PROBLEM
vehicleTracker.getVehicleLocations().get(vehicles.get(0)).setX(100); 
```

**Deep copy (safer for mutable values):**
```java
public Map<K, V> getData() {
    Map<K, V> copy = new HashMap<>();
    for (Map.Entry<K, V> e : internalMap.entrySet()) {
        copy.put(e.getKey(), new V(e.getValue())); // clone value
    }
    return copy;
}
```

### 5.4 Safe Publication
An object is safely published when both its reference and state are visible to other threads.

#### Publication Idioms
- Initialise from a static initializer
- Store reference in a volatile field or AtomicReference
- Store reference in a final field of a properly constructed object
- Store reference guarded by a lock

#### Effectively Immutable Objects
Objects that are not technically immutable but are never modified after publication.

### 5.5 Documentation
Use annotations to document thread safety:
- `@ThreadSafe`
- `@NotThreadSafe`
- `@Immutable`
- `@GuardedBy("lock")`

---

## 6. Task Execution Framework

### 6.1 Why Pooling
Starting a new thread has significant overhead (~1MB stack, ~1ms creation).
- For small tasks, overhead may predominate
- Too many concurrent threads can exhaust resources
- Pooled threads can be reused
For resources that are expensive to create, pooling is the best answer
- can limit the number of requests begin handled by queuing them up
- pooled threads can be reused indefinitely
### 6.2 Executor Framework
A task is a unit of work
ideally tasks should be:
- independent of the state of other tasks
- fine-grained (relatively small fraction of application's computation requirement)
for example single client request to a server application
in designing a concurrent system think about task not threads!
#### Executor Interface
```java
Executor executor = ...;
executor.execute(runnable);
```

#### ExecutorService
Adds lifecycle management and Future support.
```java
ExecutorService service = Executors.newFixedThreadPool(4);
Future<?> future = service.submit(runnable);
Future<Result> resultFuture = service.submit(callable);
```

#### ScheduledExecutorService
Supports delayed and periodic execution.
```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
scheduler.schedule(task, 10, TimeUnit.SECONDS);
scheduler.scheduleAtFixedRate(task, 0, 1, TimeUnit.MINUTES);
```

#### Executors Factory Methods
- `newFixedThreadPool(n)`: fixed number of threads
- `newCachedThreadPool()`: grows/shrinks with demand
- `newSingleThreadExecutor()`: single thread, replaced if dies
- `newScheduledThreadPool(n)`: for scheduled tasks
- `newVirtualThreadPerTaskExecutor()`: creates virtual thread per task (Java 21+)

### 6.3 Callable and Future
#### Callable
`Runnerable`can't return a result. What if we want to collect together the results of concurrently executing tasks?
Result-bearing version of Runnable.
```java
Callable<Integer> task = () -> {
    return computeResult();
};
```

#### Future Methods
You may want to interact with a task after it's been submitted for execution.
is it completed? Has id been cancelled? You want to cancel it!
Get the result (waiting is necessary)

`Future<V>` allows to observe and manage tasks after submission.

`Future` are created by `ExecutorService` and `CompletionService`
- you submit a `Callable<V>` to get back a `Future<V>`
- you submit a `Runnable` to get a `Future<?>`
```java
Future<Result> future = executor.submit(callable);
boolean isDone = future.isDone();
boolean isCancelled = future.isCancelled();
future.cancel(mayInterruptIfRunning);
Result result = future.get(); // blocks
Result result = future.get(timeout, TimeUnit.SECONDS); // blocks with timeout
```

#### Cancellation and Interruption
`cancel(true)` interrupts the running task; task must check for interruption.

### 6.4 CompletionService
Returns completed futures in order of completion.
```java
CompletionService<Result> cs = new ExecutorCompletionService<>(executor);
cs.submit(callable1);
cs.submit(callable2);
Future<Result> first = cs.take(); // blocks until one completes
```

### 6.5 CompletableFuture
#### Basic Usage
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return fetchData();
});
```

#### Chaining (thenApply, thenCompose, thenCombine)
```java
future
    .thenApply(data -> process(data))      // transform result
    .thenCompose(result -> fetchMore(result)) // chain another async operation
    .thenAccept(System.out::println);      // consume result
```

#### Exception Handling
```java
future
    .exceptionally(ex -> defaultValue)
    .handle((result, ex) -> {
        if (ex != null) return handleError(ex);
        return result;
    });
```

#### Combining Futures
```java
CompletableFuture.allOf(future1, future2, future3).join();
CompletableFuture.anyOf(future1, future2, future3).join();
```

### 6.6 Executor Lifecycle
#### shutdown() vs shutdownNow()
- `shutdown()`: no new tasks accepted, existing tasks complete
- `shutdownNow()`: attempts to stop all tasks, returns unexecuted tasks

#### awaitTermination()
```java
executor.shutdown();
if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
    executor.shutdownNow();
}
```

---

## 7. Fork/Join Framework
Fork-Join framework implements recursive decomposition
- repeatedly dividing a large task into smaller parts
- processing each small part independently
- joining the results together
### 7.1 Recursive Decomposition
Divide large task into smaller parts, process independently, join results.

### 7.2 RecursiveAction / RecursiveTask
To use the Fork/Join framework
#### RecursiveAction (void tasks)
define a `RecursiveAction` (for void tasks) or a `RecursiveTask<v>` (for result-bearing tasks)
```java
class MyAction extends RecursiveAction {
    protected void compute() {
        if (workload.size() > THRESHOLD) {
            ForkJoinTask.invokeAll(createSubtasks());
        } else {
            process(workload);
        }
    }
}
```

#### RecursiveTask (result-bearing)
 template for `RecursiveAction`: to override `compute`
```java
class MyTask extends RecursiveTask<Integer> {
    protected Integer compute() {
        if (size <= THRESHOLD) {
            return computeDirectly();
        }
        MyTask left = new MyTask(leftHalf);
        MyTask right = new MyTask(rightHalf);
        left.fork();
        int rightResult = right.compute();
        int leftResult = left.join();
        return leftResult + rightResult;
    }
}
```

### 7.3 Work Stealing Algorithm
Idle threads steal tasks from busy threads' queues, improving load balancing.

### 7.4 Parallel Streams
```java
list.parallelStream()
    .filter(x -> x > 0)
    .map(x -> x * 2)
    .sum();
```

Conditions for effective parallel streams:
- Computationally intensive workload
- Per-element independent operations
- Sequential version takes ~50+ microseconds
- Efficiently splittable source (ArrayList yes, LinkedList no)
- Sufficient cores to outweigh overhead

---

## 8. Concurrent Collections

### 8.1 Problems with Standard Collections
ArrayList, HashSet, HashMap are not thread-safe.

### 8.2 Synchronized Wrappers
```java
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
```
Limitations:
- Client-side locking needed for compound operations
- Risk of ConcurrentModificationException during iteration
- Performance bottleneck (single lock for all operations)

### 8.3 ConcurrentHashMap
#### Features
- Concurrent read operations
- Thread-safe atomic operations
- Safe iteration (weakly consistent)
- No client-side locking needed

#### Atomic Operations
```java
map.putIfAbsent(key, value);
map.remove(key, expectedValue);
map.replace(key, oldValue, newValue);
map.computeIfAbsent(key, k -> createValue(k));
map.computeIfPresent(key, (k, v) -> updateValue(v));
map.compute(key, (k, v) -> newValue);
map.merge(key, value, (old, new) -> combine(old, new));
```

### 8.4 CopyOnWriteArrayList/Set
Copy entire array on each modification.
- Good for read-heavy, write-rare scenarios
- Iteration never throws ConcurrentModificationException
- High write cost

### 8.5 ConcurrentSkipListMap/Set
Concurrent sorted map/set based on skip lists.
- O(log n) operations
- Sorted iteration

### 8.6 Blocking Queues
#### ArrayBlockingQueue
Bounded, array-backed, fair ordering optional.

#### LinkedBlockingQueue
Optionally bounded, linked-node backed.

#### PriorityBlockingQueue
Unbounded, elements ordered by priority.

#### DelayQueue
Elements become available after a delay.

### 8.7 Deques
#### ConcurrentLinkedDeque
Non-blocking concurrent deque.

---

## 9. Synchronizers

### 9.1 Semaphore
Manages a finite set of permits.
```java
Semaphore sem = new Semaphore(3); // 3 permits
sem.acquire(); // blocks if no permits
try {
    // use resource
} finally {
    sem.release();
}
```

### 9.2 CountDownLatch
One-time gate; opens when count reaches zero.
```java
CountDownLatch latch = new CountDownLatch(3);

// Worker threads:
latch.countDown();

// Main thread:
latch.await(); // blocks until count is 0
```

### 9.3 CyclicBarrier
Reusable barrier; threads wait until all arrive.
```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> {
    // action when all threads arrive
});

// Each thread:
barrier.await(); // blocks until all threads call await()
```

### 9.4 Phaser
Flexible synchroniser for variable number of parties across multiple phases.

### 9.5 Exchanger
Two threads exchange objects at a synchronisation point.
```java
Exchanger<Buffer> exchanger = new Exchanger<>();

// Thread 1:
Buffer full = fillBuffer();
Buffer empty = exchanger.exchange(full);

// Thread 2:
Buffer empty = new Buffer();
Buffer full = exchanger.exchange(empty);
```

### 9.6 SynchronousQueue
Queue with no capacity; each put must wait for a take.

---

## 10. Virtual Threads (Java 21+)

### 10.1 Motivation
Reactive programming handles IO blocking but has complex programming model.
Virtual threads offer same Thread API with very low overhead.

### 10.2 Platform vs Virtual Threads
| Aspect | Platform Thread | Virtual Thread |
|--------|-----------------|----------------|
| Stack size | ~1MB | ~few KB |
| Creation cost | ~1ms | ~microseconds |
| Max count | ~thousands | ~millions |
| Scheduling | OS preemptive | JVM non-preemptive |

### 10.3 Carrier Threads
Virtual threads run on a small pool of platform threads (carrier threads).

### 10.4 Scheduling Model
- OS schedulers are preemptive (time-slicing)
- Virtual threads are non-preemptive (run until block)

### 10.5 Pinning
Virtual thread cannot unmount from carrier when:
- Inside synchronized block/method
- Executing native method

Use ReentrantLock instead of synchronized to avoid pinning.

### 10.6 When to Use
- IO-intensive workloads: YES
- CPU-intensive workloads: NO (use platform threads/parallel streams)

```java
// Creating virtual threads
Thread vt = Thread.ofVirtual().start(() -> task());

// Using executor
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(task);
}
```

---

## 11. Structured Concurrency (Preview)

### 11.1 StructuredTaskScope
Treats multiple tasks as a single unit of work.

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Subtask<String> subtask1 = scope.fork(callable1);
    Subtask<Integer> subtask2 = scope.fork(callable2);

    scope.join();
    scope.throwIfFailed();

    return combine(subtask1.get(), subtask2.get());
}
```

### 11.2 ShutdownOnSuccess
Completes when first subtask succeeds; cancels remaining.

```java
try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
    scope.fork(callable1);
    scope.fork(callable2);

    scope.join();
    return scope.result(); // first successful result
}
```

### 11.3 ShutdownOnFailure
Completes when all subtasks succeed or one fails.

### 11.4 Custom Policies
Extend StructuredTaskScope to implement custom completion policies.

---

## 12. Concurrency Patterns

### Producer-Consumer
Producers add to queue, consumers take from queue.
```java
BlockingQueue<Task> queue = new LinkedBlockingQueue<>();

// Producer
queue.put(task);

// Consumer
Task task = queue.take();
```

### Reader-Writer
Multiple readers OR single writer.
Use ReadWriteLock or StampedLock.

### Thread-per-Task
Simple model; each task gets new thread.
Not scalable; use thread pools instead.

### Thread Pool
Reuse fixed set of threads for multiple tasks.

### Poison Pill
Special value signals consumer to shutdown.
```java
queue.put(POISON_PILL);

// Consumer
Task task = queue.take();
if (task == POISON_PILL) return;
```

---

## 13. Debugging and Testing

### Thread Dumps
Capture thread state for analysis.
```bash
jstack <pid>
kill -3 <pid>  # on Unix
```

### Deadlock Detection
JVM can detect monitor deadlocks in thread dumps.
Use `ThreadMXBean` for programmatic detection.

### Testing Concurrent Code
- Use stress testing with many threads
- Use tools like jcstress for concurrency correctness
- Consider Thread.sleep() or CountDownLatch to control timing
- Use assertions and invariant checks

### Common Pitfalls
- Racing on compound operations (check-then-act)
- Publishing partially constructed objects
- Double-checked locking without volatile
- Inconsistent lock ordering (deadlock)
- Calling alien methods while holding locks

---

## References
- Java Concurrency in Practice (Brian Goetz)
- Java Memory Model - JSR 133
- Java Generics and Collections
- Mastering Lambdas (Maurice Naftalin)



