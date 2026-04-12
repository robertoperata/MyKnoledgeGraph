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
**JNI (Java Native Interface) boundaries** refer to the transition points between Java code and native (C/C++) code. From a GC perspective, these are critical because:

- Native code can hold references to Java objects
- The GC needs to know about these references to avoid collecting objects that native code is using
- Special handling is required for objects passed across the JNI boundary
- JNI references can be "pinned" in memory, affecting GC movement algorithms# JNI