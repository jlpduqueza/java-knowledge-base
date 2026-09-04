# The Java Collections Framework

A unified architecture for representing and manipulating collections of objects (`java.util`).

## Core Interfaces

- **Collection** — root interface: `List`, `Set`, `Queue`.
- **List** — ordered, allows duplicates, index-based access (`ArrayList`, `LinkedList`).
- **Set** — no duplicates (`HashSet`, `LinkedHashSet`, `TreeSet`).
- **Queue / Deque** — FIFO/LIFO structures (`ArrayDeque`, `PriorityQueue`).
- **Map** — key/value pairs, not a true `Collection` (`HashMap`, `LinkedHashMap`, `TreeMap`). 

## Choosing an Implementation

| Need                                  | Use            |
|----------------------------------------|----------------|
| Fast random access by index            | `ArrayList`    |
| Frequent insert/remove at ends/middle  | `LinkedList`   |
| No duplicates, fast lookup             | `HashSet`      |
| No duplicates, insertion order kept    | `LinkedHashSet`|
| No duplicates, sorted order            | `TreeSet`      |
| Key-value lookup                       | `HashMap`      |
| Key-value, insertion order kept        | `LinkedHashMap`|
| Key-value, sorted by key               | `TreeMap`      |
| Stack/queue behavior                   | `ArrayDeque`   |
| Priority-ordered processing            | `PriorityQueue`|

## Big-O Cheat Sheet

| Operation        | ArrayList | LinkedList | HashMap | TreeMap |
|-------------------|:---------:|:----------:|:-------:|:-------:|
| get by index      | O(1)      | O(n)       | -       | -       |
| get by key        | -         | -          | O(1)*   | O(log n)|
| add/remove at end | O(1)*     | O(1)       | -       | -       |
| add/remove at head| O(n)      | O(1)       | -       | -       |
| contains          | O(n)      | O(n)       | O(1)*   | O(log n)|

\* amortized / average case; hash collisions can degrade `HashMap` toward O(n) in the worst case.

## Iteration and Fail-Fast Behavior

Most collections are fail-fast: structural modification during iteration (outside the iterator's own `remove()`) throws `ConcurrentModificationException`.

```java
List<String> names = new ArrayList<>(List.of("a", "b", "c"));
Iterator<String> it = names.iterator();
while (it.hasNext()) {
    String name = it.next();
    if (name.equals("b")) {
        it.remove(); // safe - goes through the iterator
    }
}
```

## Immutable and Unmodifiable Collections

```java
List<Integer> immutable = List.of(1, 2, 3);        // truly immutable
List<Integer> unmodView = Collections.unmodifiableList(list); // view; backing list can still change
```

## Sorting and Comparators

```java
List<Person> people = new ArrayList<>(...);
people.sort(Comparator.comparing(Person::getLastName)
                       .thenComparing(Person::getFirstName));
```

## equals()/hashCode() Contract for Keys

Objects used as `HashMap`/`HashSet` keys must implement `equals()` and `hashCode()` consistently — equal objects must produce equal hash codes, or lookups will silently fail to find entries that "should" match.

## Concurrent Collections

For multi-threaded access prefer `java.util.concurrent` types over synchronizing manually:
- `ConcurrentHashMap` — thread-safe map with fine-grained locking.
- `CopyOnWriteArrayList` — safe for many reads, rare writes.
- `BlockingQueue` implementations — producer/consumer pipelines.
