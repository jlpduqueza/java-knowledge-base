# JVM Internals

How Java code actually runs, at a level useful for debugging performance and memory issues.

## From Source to Execution

1. `.java` source is compiled by `javac` into `.class` bytecode.
2. The JVM's **class loader** loads classes on demand (bootstrap, platform, and application class loaders, in a delegation hierarchy).
3. Bytecode is interpreted at first, then hot methods are compiled to native machine code by the **JIT compiler** (C1/C2 tiers in HotSpot) for speed.

## Runtime Memory Areas

| Area           | Holds                                         | Notes |
|-----------------|-----------------------------------------------|-------|
| **Heap**        | All objects and arrays                        | Shared across threads; garbage collected |
| **Stack**       | Local variables, method call frames            | One per thread; stack overflow if too deep |
| **Metaspace**   | Class metadata (replaced PermGen in Java 8+)   | Grows in native memory, not the heap |
| **PC Register** | Address of the currently executing instruction | One per thread |
| **Native Stack** | Native (JNI) method calls                     | One per thread |

The heap is further divided (implementation-specific) into a **young generation** (Eden + Survivor spaces) for new objects and an **old generation** for long-lived objects, reflecting the generational hypothesis: most objects die young.

## Garbage Collection Basics

The GC automatically reclaims memory for objects no longer reachable from GC roots (local variables, static fields, active threads).

- **Minor GC** — collects the young generation; frequent, usually fast.
- **Major/Full GC** — collects the old generation (or everything); less frequent, can pause the application longer.
- Objects that survive several minor GCs are **promoted** to the old generation.

### Common Collectors (HotSpot)
- **G1 (Garbage First)** — default since Java 9; balances throughput and pause time, region-based.
- **ZGC** / **Shenandoah** — designed for very low pause times (sub-millisecond), even on large heaps.
- **Parallel GC** — optimizes for throughput over pause time.

## Common JVM Flags

```
-Xms512m -Xmx2g        # initial and max heap size
-XX:+UseG1GC            # select a garbage collector
-XX:MetaspaceSize=128m  # metaspace sizing
-Xss512k                # thread stack size
```

## Memory Leaks in a Garbage-Collected Language

Java can still leak memory when objects stay reachable longer than intended:
- Static collections that grow forever (a cache with no eviction).
- Listener/observer registrations that are never removed.
- Long-lived objects holding references to short-lived ones (e.g. an outer class implicitly held by a non-static inner class).
- `ThreadLocal` values not cleared, especially in pooled-thread environments.

## Escape Analysis and Stack Allocation

The JIT can sometimes prove an object never "escapes" a method (no reference leaves it), allowing it to allocate that object on the stack — or avoid allocating it at all — instead of the heap, reducing GC pressure. This is an internal optimization; you cannot force it, but writing methods that don't leak references to their local objects makes it more likely.
