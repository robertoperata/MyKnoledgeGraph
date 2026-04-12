---
tags:
  - java
  - jvm
  - gc
feature:
type:
author: "[[Kirk Pepperdine]]"
source: "[[Java GC Tuning]]"
---
# Mutators

In GC terminology, **mutators** are the application threads that modify (mutate) the object graph during program execution. Essentially, any code that:

- Creates new objects
- Updates object references
- Deletes references

The term comes from the fact that these threads "mutate" the heap structure while the garbage collector is trying to manage it. This creates the classic concurrency challenge in GC design - how to collect garbage while mutators are actively changing what needs to be collected.

## JNI Boundaries

**JNI (Java Native Interface) boundaries** refer to the transition points between Java code and native (C/C++) code. From a GC perspective, these are critical because:

- Native code can hold references to Java objects
- The GC needs to know about these references to avoid collecting objects that native code is using
- Special handling is required for objects passed across the JNI boundary
- JNI references can be "pinned" in memory, affecting GC movement algorithms