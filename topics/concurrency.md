# Concurrency in Java

## Threads: The Basics

```java
Thread t = new Thread(() -> System.out.println("running"));
t.start();   // starts a new thread - never call run() directly for that
t.join();    // wait for it to finish
```

Prefer `Runnable`/`Callable` + an `ExecutorService` over managing raw `Thread` objects yourself.

## ExecutorService

```java
ExecutorService pool = Executors.newFixedThreadPool(4);
Future<Integer> future = pool.submit(() -> computeSomething());
Integer result = future.get(); // blocks until done
pool.shutdown();
```

Common factory methods: `newFixedThreadPool(n)`, `newCachedThreadPool()`, `newSingleThreadExecutor()`, `newVirtualThreadPerTaskExecutor()` (Java 21+, lightweight virtual threads).

## The Java Memory Model (JMM)

Defines when changes made by one thread are guaranteed visible to another. Without synchronization, the compiler/CPU/cache may reorder or cache operations, so a value written by one thread might never become visible to another.

- **`synchronized`** — mutual exclusion + a happens-before edge (lock release happens-before the next acquire).
- **`volatile`** — guarantees visibility of a single variable across threads (writes are flushed, reads are fresh), but does not provide atomicity for compound actions like `count++`.
- **`final`** fields, safely published, are visible to other threads without extra synchronization once construction completes.

## synchronized

```java
public class Counter {
    private int count = 0;

    public synchronized void increment() { count++; }
    public synchronized int get() { return count; }
}
```

Only one thread can execute a `synchronized` method/block on a given object's monitor at a time. Over-synchronizing hurts throughput; under-synchronizing causes race conditions — synchronize the smallest critical section that keeps invariants correct.

## Atomic Classes

`java.util.concurrent.atomic` provides lock-free, thread-safe operations on single variables:

```java
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();       // atomic, no synchronized needed
counter.compareAndSet(5, 10);    // CAS: compare-and-swap
```

## Common Concurrency Utilities

| Class                  | Purpose                                            |
|-------------------------|-----------------------------------------------------|
| `CountDownLatch`        | Block until N events have occurred                 |
| `CyclicBarrier`         | Wait for a fixed set of threads to all reach a point|
| `Semaphore`             | Limit concurrent access to a resource               |
| `ReentrantLock`         | Explicit lock with more control than `synchronized`|
| `ConcurrentHashMap`     | Thread-safe map, fine-grained locking               |
| `BlockingQueue`         | Producer/consumer handoff, blocks when full/empty  |
| `CompletableFuture`     | Compose async computations without blocking         |

## CompletableFuture

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> fetchData())
    .thenApply(data -> transform(data))
    .exceptionally(ex -> "fallback");

future.thenAccept(System.out::println);
```

## Deadlocks and How to Avoid Them

A deadlock happens when two or more threads each hold a lock the other needs and wait forever. Common cause: acquiring multiple locks in inconsistent order.

- Always acquire locks in a fixed, global order.
- Prefer higher-level utilities (`ExecutorService`, `java.util.concurrent` collections) over hand-rolled locking.
- Use lock timeouts (`tryLock(timeout)`) where deadlock risk is real.

## Virtual Threads (Java 21+)

Lightweight threads managed by the JVM rather than mapped 1:1 to OS threads, making it cheap to run millions of blocking-style tasks concurrently:

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> handleRequest());
}
```
