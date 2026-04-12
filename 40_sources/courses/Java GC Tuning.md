---
tags:
  - java
feature:
type:
author: "[[Kirk Pepperdine]]"
source: https://learning.oreilly.com/live-events/java-gc-tuning/0642572188467/
---
# Tuning Java Automated Managed Memory


>[!iimportant] Take away:
>1. what does the cost model look like for garbage collection?
>2. what are the inputs to that cost model?
How can I use that to help me:
>1. understand which collector I want to use
>2. what will be the inpact of tuning this of any collector
## Brief history of GC
- STW (stop the world): **single thread** access the "*single memory pool*". So either the CG or the application can access memory (**mutual exclusion**). Definition of a **[[Java GC Tuning#Safe-point|safe-point]]**.
- STW (stop the world): **multiple thread** access the "*single memory pool*". So either the CG or the application can access memory (**mutual exclusion**). Definition of a **[[Java GC Tuning#Safe-point|safe-point]]**.
- enabled GC to run concurrently with the application threads. Needed to define a **GC barrier** so that the GC threads and the application threads access the GC barrier to understand how to cooperate with each other while they're cleaning Java heap
- **Generational spaces** (based on the [[Java GC Tuning#Tuning Java Automated Managed Memory#Weak Generational Hypothesis|weak generational hypothesis]]) employ a couple of different strategies for collecting memory. It takes the *single memory pool* and splits it into **old space** and young space. The young space is split in **eden space** and **survivor space**. These segments of Java heap separate data by age.

## Weak Generational Hypothesis

^d4b62c

- the vast majority of data is live for a very brief period of time
- the longer data is live the more likely it will remain live

## Latency
$\text{latency} = \text{ time response} - \text{time input}$
Given a set of response times sorted by time:
41, 42, 42, 43, 44, 46, 46, 73, 79, 93, 112
$pX$ is the value where $X\%$ of the numbers are greater and $100-X\%$ are smaller
p50 = 46 (the latency at 50% of the response)
p90 = 93 (the latency at 90% of the response)

There are many different types of Garbage Collectors.

The "Concurrent Collector" impacts across all responses (increasing latency) - p50, p10, p80 because the concurrent Collector needs an hand shaking protocol with the application threads in order to give you a **consistent view of memory**. That impacts:
- collector uses CPU and accessing memory while the application uses CPU and accessing memory
- while running we have to invoke code in order to make sure that everything looks of, taht if somebody comes along and changes a value or moves data that everbody else knows that this happens.

If you are measuring latency <u>inside</u> the JVM, that latency may miss the impact of GC pause

### What “tail latency” means (in general)

**Tail latency** refers to the _worst-case response times_ in a system — the “slowest few” requests or operations.

- For example:
    - **Average latency** might be 20 ms,
    - but the **99th percentile latency** (the “tail”) might be 500 ms.
- This means 99% of requests finish in 20 ms, but 1% are very slow — that’s the _tail_.
   
So, **tail latency** describes those rare but **painfully slow outliers** that affect user experience, especially in real-time or high-throughput systems.

**In the context of the *Java Garbage Collector*, tail latency refers to long, *unpredictable GC pauses* that affect response times.**

## Safe-point
**is a point in threads execution where the state is well understood**. So I can stop the thread at this point and I can manipulate all these data structures, including memory, and then I can release the application threads again and they will function without any issues.
Dividing "things can wait" and "things that have to be done now", GC is an on-demand "things to be done now" type of operation.
On a high level view when GC kicks in and take all the application threads to safepoint. Solving the code with safepoint checks. The safepoint check is just to check a [[Reserve Page|reserve page]]

- code is salted with Safepoint checks
	- [[Mutators]] poll the safepoint ([[Reserve Page|reserve page]])
		- on back edge of non-counted loops
		- method exit
		- [[JNI]] boundaries
- Implicitly at a a safepoint when
	- waiting on a [[Monitors|monitor]]
	- blocked on IO
	- in primitive or [[JNI]] code


Maintenance will be delayed until all the application thread are in a safepoint. and sometimes *can take much longer to threads to reach the safepoint than the JVM do the maintenece work*
And this means that the mutator threads that have reached the safepont are blocked until also the other threads reach the safepoint

### Counted Loops vs Non-Counted Loops
**Counted Loops**
```java
for (int i = 0; i < 1000; i++) {
	toatl += I
}

while (i < 1000) {
	total += i;
	i++;
}
```
**NO SAFEPOINT CHECK**

**Non-Counted Loops**
```java
for (int i = 0; i < 1000; i+=2) {
	total += I;
}

while (i++ < 1000) {
	total += i;
}
```
**SAFEPOINT CHECK ON BRANCH BACK**

to analyse memory use https://github.com/async-profiler/async-profiler
>[!tip] Print the safepoint statistics then see how many nanoseconds take for all threads to reach the safepoint.

### Java Heap
By default java allocates to the heap 1/4 of the host memory or in CGroup (container) for the "Virtual Memory" that is backed by the physical ram

-Xmx = 1/4 RAM (max heap size)
-Xms = 1/64 RAM (min heap size) - initial heap size 1/16

than create generational spaces into young space and old space.
The mechanics is different for each GC
For the serial parallel GC it's going to reserve a ONE contiguous and then remap in two contiguous chunk for young and old generation
G1, ZGC in Shenandoah reserves the entire chunk, the max heap size chunk
#### G1
G1 is a regional collector = creates the virtual memory space but do not divide into a young and old instead divides into 2048 regions. So it takes the heap and size the region to have almost 2048 and each region could be of 1,2,4,8,16 o 32 megabytes
the young generation (Eden) are put from left
the old generation (Tenured) are put from right

With Serial and Parallel collector we start collection on allocation failure. When we have a concurrent collector we don't want till we get an allocation failure because we have to stop the applications. We want the garbage collector to run concurrently with the application, which means that I'm going to wait till it gets to a certain level of fullness and when the heap gets that full then that's when we are going to start a collection style. I'm hoping for that the garbage collector cycle will finish before heap fills. The metric to look in this case is the allocation rate, because the allocation rate says how fast we are filling and this gives the time to the garbage collector to finish to free memory.
If the heap fills during a concurrent cycle it's know as concurrent failure and there's different responses to concurrent failure depending upon the current collector that we are using.
With G1 we get a Full Stop The World collection (full GC)
#### Java Heap ZGC
It looks like G1, the only differences is that we are going to have small pages (regions), medium pages (regions) and large pages (regions). and to distinguish from young and old the the pages are tagged
### Allocation cycle
The allocation are going to be generally a "bump and run" on top of heap pointer.
the **bump pointer allocation**:
1. **The heap maintains a pointer** to the next available memory location
2. **To allocate memory**: Simply move ("bump") the pointer forward by the size of the object
3. **No searching required**: No need to find free space or maintain free lists
The Bump and Run Process:
- **Check**: Is there enough space between the current pointer and the end of the region?
- **Allocate**: If yes, place the object at the current pointer location
- **Bump**: Move the pointer forward by the object size
- **Run**: Continue with program execution
If we just have a top of heap pointer and all the allocations are chasing that thing, then we have a problem called we have a problem because a single data point that is going to be written by multiple threads.
there are other data strucutre that take off pressure off from the top of heap pointer.
the **Thread Local Allocation Block (TLAB)** 
TLAB: help partition heap and help manage heap. Mapping on top of Java heap. and in particular handles the Eden (young) space.
```java
//Thread 1
Foo foo = new Foo()
```
on the `new` operations is going to say: I need to go into the heap and I need a chunk of memory. Thread 1 where's your TLAB? or if you don't have one no problem. we are going to do a global allocation of a TLAB for you.
TLAB is a chunk of memory which the size change depending upon how memory active the thread is.
After `Foo foo = new Foo();` is going to grab the TLAB and set another pointer top of TLAB.
Only thread 1 can allocate into that particular TLAB. That's get rid of a lot of the concurrency issues and takes a top of pressure off of the top of the heap pointer.
than I take the amount of memory that I need to allocate `Foo foo = new foo();` and "bump and run" on top of the TLAB pointer and than I put the data for Foo right in that spot.
This is repeated for Thread 2 with another TLAB that is accessible (writeable) only by thread 2.
TLAB are writeable only by one thread but they may be red by other treads (getter or public)
in case an Object larger than the amount of memory space of the TLAB is set directly to the heap. similar to a flyway pattern.
TLAB is kind of a view of the eden space
As soon as I get pass the **waste percentage** than I can get another TLAB I continue on allocating, but untill I go over that threshold, if I don't have enough space in the TLAB, I'm going to allocate outside of the top of the heap.
>[!tip] sometime adjusting the waste percentage is helpful 

Allocation greater than half of the Eden will be allocated directly in tenured (old space)
>[!question] Is tenured a parallel serial collector

Allocations failures in Eden cause a GC cycle, so that's good for parallel serial and G1.
>[!question] Why allocations failures in Eden cause a GC cycle are good for parallel, serial and G1?

allocations failures means don't have enough space in the TLAB

>[!attention] Check different JVM implementation. Zing JVM?

## Tenured Allocations
All the allocations in tenured are performed by the garbage collection thread because they're coping data from young Gen into tenured space and they're only copy the data that is live so the thing that is surviving the garbage collections and has reached a tenuring threshold.
>[!important] this threshold is super important because it's tuneable and if you get that right you can reduce GC overhead to incredibly small amount, and if you get it wrong could be very messy

each GC thread gets it's own PLAB (Promotional Local Allocation Blocks)
when we get into Eden, we are going to use an evacuating collector for Young generation but it's an in place collection in tenured space and the mechanics of these are very different.
You don't have to worry about memory fragmentation in young gen but you have to warry about memory fragmentation in the tenured gen (much less with regional collectors, but more with serial, parallel collectors because if they are heavely fragmented they also have to perform a compaction

## G1 Heap Allocations
Humongous regions menage allocations that are larger that the size of the region

## Garbage Collection Cycle
- precies (tracing) Mark & Seep collectors
	- find all object that are reachable from GC root set
	- GC roots are objects that are live by definition
	- 
we start from the GC root set and we're going to performa a reachability test. I'm just going to cascade that out until I hit the feaf node in my object graph - I can't go any further.
That marks the data as being live and then that allows me to sweep taht data and in the process a can do a couple of things:
- if I have a copying collector then I can move that data into another space
- if I have in place collector than I try to look at the memory that hasn't marked and return that to a free list.
one of these collectors triggered, for the parallel serial collector and G1 and when there's concurrent failures I'm going to trigger these type of collection when the memory pool is filled.
for a concurrent collector I'm going to have to use a heuristic, and the heuristic is going to say when I get a certain level of fullness start the collection cycle because it needs to finish before I get an allocation failure.

### GC Roots
in the language of garbage collectors we talk about two different types of pointers.
we have an internal pointer and an external pointer.
the internal pointer is going to have the source and the destination of the pointer. source and destination both pointing within the same memory pool
the external pointer have the source external to the memory pool that I'm interested in collecting.
So the first things we want to do when we want to collect a memory space is to find all the external pointers that's going to form my root set
There are many GC roots:
- statics
- metaspace
- compressed classpace
- code cache
- stack frames
- registers
- other memory pools
>[!hint] one thing that inflate the pause time of the garbage collector is the number of threads that I have in my JVM. 
>1. It takes much more time to get 5000 threads to a safe point than it does to get like 2 (Longer TTSP - time to safe point)
>2. but also more stack frames that I have to scan for GC roots
>That's could happen when you use a technology on the front memory, like thread pools for incoming requests, thread pools for connection pools, threads going out back end. So for example have a 200 hundreds thread on a kubernetes 4 cores

our external pointers are going to form a root set. we are going to have an initial mark, and that's what it's going to use for concurrent collectors is going to scan the roots set and it's going to internalise all the external pointers
## Tactics
for in-place collections I need to use a free list to track free memory chunks and I'm going to return unused memory of freed memory back to the free list.

for evacuating copying collectors data is going to be copied to a "to-space"

The cost model of a in place collection is going to be very different from the cost model of a copying collector.
### Place Collectors
the **in place collectors** I need to manage the free list all the time and I need to worry about 
fragmentation because I might to do a compaction.
### Coping collectors
In the **coping collectors** I don't have fragmentation issue. I don't have a lot of issue in terms of like managing space, these extra data structures like free list. I'm just pushing over to another space, I'm able to compact and the free list is just to be the top of heap pointer. Requires more memory and have a copy costs. but in general it's cheaper simply because the [[Java GC Tuning#^d4b62c|weak generational hypothesis]]. So if I employ thie young gnerational space where I'm just going to copy the live data out, I know from the [[Java GC Tuning#^d4b62c|weak generational hypothesis]] that the vast majority of data in young space is not going to be live and the cost of this collection is going to be related to the number of live objects. so most of the data is not live this generally turns out to  be a very cheap implementation much cheaper that doing the in place collection with the free list.

if G1 collector is properly tuned the major cost should be the copy cost. And should be **at least 90% dominant** (if not more).

There's some issues with the current collector that you don't see with the serial and the parallel collector where you have the full stop the world behavior knowing as floating garbage. things that we can not identify as been dead. So we treated live

## Garbage Collection Phases


## Context — concurrent GCs and "handshakes"

In a **concurrent collector**, the GC runs **alongside your application threads** (as opposed to _stop-the-world_ collectors, which pause everything).

That’s great for latency, because your app doesn’t freeze for long pauses.

But…  
there’s a big **consistency problem**:

>[!question] If the GC is scanning or relocating objects while your program is _mutating_ them, how can both see a consistent view of memory?

####  “While running we have to invoke code …”

This means that while the application threads are running, the GC **injects small bits of coordination code** into their normal execution — to keep memory consistent.

#### Example situation

1. The GC is scanning the heap, marking which objects are still reachable.
2. At the same time, your application **changes a reference** — e.g.:
    ```java
    myObject.field = anotherObject;
    ```
  
3. If GC doesn’t know that reference changed, it might think the old object is garbage and reclaim it too early. 💥
#### Solution: "barriers" and "handshakes"

To prevent that, concurrent GCs use **barriers** — small bits of code that run when you read or write object references.
There are two kinds:
##### **Write Barrier**
Runs when a thread _writes_ a reference.
- It tells the GC: “Hey, this reference changed!” 
- The GC can then update its metadata (mark bits, remapping tables, etc.)
- Example: _Card marking_ in G1 GC.
##### **Read Barrier**
Runs when a thread _reads_ a reference.
- Used in GCs that move objects concurrently (like **ZGC**, **Shenandoah**). 
- It checks whether the object was relocated and fixes the pointer if needed (the so-called **colored pointers** or **load barriers**).
#### So what the sentence means

>[!quote] “While running we have to invoke code to make sure that everything looks OK, that if somebody comes along and changes a value or moves data that everybody else knows that this happens.”

- The GC inserts extra _bookkeeping operations_ into normal program execution (the “invoke code” part).
- This ensures both the GC and the running app have a **consistent shared understanding** of where objects live and what references point where.
- These safety checks make sure:
	- No object is freed while it’s still in use,
    - No thread reads stale or moved references.

But those checks **cost CPU cycles and memory bandwidth**, which is why **concurrent collectors increase average latency (p50, p80)** — even if they reduce the worst-case pauses.
#### In summary

| Concept                       | What it means                                                        | Cost                     |
| ----------------------------- | -------------------------------------------------------------------- | ------------------------ |
| **Handshakes**                | Points where GC coordinates with running threads                     | Small context switches   |
| **Barriers**                  | Extra code injected into reads/writes to maintain memory consistency | Increases CPU usage      |
| **Consistent view of memory** | GC and app see the same live object graph                            | Requires synchronization |
| **Trade-off**                 | Lower tail latency but slightly higher average latency               | True for ZGC, Shenandoah |



![[hotspotgarbagecollection1752462947432.pdf]]