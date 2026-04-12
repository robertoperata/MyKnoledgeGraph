---
tags:
  - jvm
  - gc
  - java
feature:
type:
author: "[[Kirk Pepperdine]]"
source: "[[Java GC Tuning]]"
---
# Reserve Page
The **reserve page** (also called a **guard page** or **polling page**) is a special memory page used as an efficient mechanism to implement safepoint checks in the JVM.

## How it Works

1. **Normal Operation**: The JVM maps a memory page that application threads periodically "poll" by attempting to read from it. Initially, this page is mapped as readable, so the polling operation succeeds quickly.
2. **Safepoint Request**: When the GC needs all threads to reach a safepoint, the JVM unmaps or changes the protection of this reserve page to make it unreadable/inaccessible.
3. **Thread Suspension**: When application threads next attempt their safepoint poll (reading from this page), they trigger a page fault/segmentation fault.
4. **Signal Handling**: The JVM's signal handler catches this fault and realizes it's a safepoint poll on the now-protected page. Instead of crashing, it suspends the thread at this safepoint.

## Why This Design?

This approach is much more efficient than alternatives because:

- **Fast Path**: During normal execution, safepoint polling is just a simple memory read - very fast
- **No Branches**: No conditional checks needed in the common case
- **Hardware Assisted**: Uses the OS/hardware memory protection mechanism rather than software polling

## The "Check"

The safepoint check the author mentions is typically compiled into loops and method calls as something like:

```assembly
movl (%rsi), %eax  ; Read from the polling page
```

When this instruction hits the protected reserve page, it triggers the safepoint mechanism.

This is a clever use of memory protection hardware to create an efficient global thread coordination mechanism - much faster than having threads constantly check a flag variable.