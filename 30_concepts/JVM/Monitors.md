---
tags:
  - jvm
  - java
  - gc
feature:
type:
author: "[[Kirk Pepperdine]]"
source: "[[Java GC Tuning]]"
---
**Monitors** in this context refer to Java's synchronization primitives - essentially the implementation of `synchronized` blocks/methods. They're relevant to GC because:

- Monitor metadata is often stored in object headers
- Lock inflation/deflation affects object layout
- Biased locking optimizations can complicate object movement during GC
- The GC needs to preserve synchronization state when moving objects# Monitors