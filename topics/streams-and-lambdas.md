# Streams and Lambdas (Java 8+)

## Lambda Expressions

A lambda is a concise way to implement a functional interface (an interface with exactly one abstract method).

```java
// Before: anonymous class
Comparator<String> byLength = new Comparator<String>() {
    public int compare(String a, String b) {
        return a.length() - b.length();
    }
};

// After: lambda
Comparator<String> byLength = (a, b) -> a.length() - b.length();
```

## Common Functional Interfaces (java.util.function)

| Interface           | Method             | Purpose                          |
|----------------------|---------------------|-----------------------------------|
| `Function<T, R>`     | `R apply(T t)`      | Transform T into R                |
| `Predicate<T>`       | `boolean test(T t)` | Test a condition                  |
| `Consumer<T>`        | `void accept(T t)`  | Do something with T, return nothing|
| `Supplier<T>`        | `T get()`           | Produce a T with no input         |
| `BiFunction<T,U,R>`  | `R apply(T t, U u)` | Transform two inputs into R       |

## Method References

A shorthand for a lambda that just calls an existing method:

```java
names.forEach(System.out::println);      // instance method on argument
list.stream().map(String::toUpperCase);  // instance method on argument
list.stream().map(Object::toString);     // instance method reference
Stream.generate(Math::random);           // static method
names.stream().map(Person::new);         // constructor reference
```

## The Stream API

Streams describe a pipeline of operations over a data source; they are lazy and don't run until a terminal operation is invoked. A stream can only be consumed once.

```java
List<String> result = people.stream()
    .filter(p -> p.getAge() >= 18)
    .map(Person::getName)
    .sorted()
    .collect(Collectors.toList());
```

### Intermediate Operations (lazy, return a Stream)
`filter`, `map`, `flatMap`, `sorted`, `distinct`, `limit`, `skip`, `peek`

### Terminal Operations (trigger execution)
`collect`, `forEach`, `reduce`, `count`, `anyMatch`/`allMatch`/`noneMatch`, `findFirst`/`findAny`, `toArray`

## Collectors

```java
Map<Boolean, List<Person>> byAdult = people.stream()
    .collect(Collectors.partitioningBy(p -> p.getAge() >= 18));

Map<String, List<Person>> byCity = people.stream()
    .collect(Collectors.groupingBy(Person::getCity));

String joined = people.stream()
    .map(Person::getName)
    .collect(Collectors.joining(", ", "[", "]"));
```

## reduce()

```java
int total = numbers.stream()
    .reduce(0, (a, b) -> a + b); // identity + accumulator
```

## Parallel Streams

`stream()` can become `parallelStream()` to use the common `ForkJoinPool` across multiple cores. Useful only for large datasets with CPU-bound, stateless, non-blocking operations — the coordination overhead can make it slower for small inputs.

## Optional

Represents a value that may or may not be present, discouraging `null` checks:

```java
Optional<Person> found = people.stream()
    .filter(p -> p.getId() == targetId)
    .findFirst();

String name = found.map(Person::getName).orElse("unknown");
```

## Common Pitfalls

- Reusing a stream after a terminal operation throws `IllegalStateException`.
- Using streams with side effects (mutating external state inside `map`/`filter`) instead of pure functions — hurts readability and breaks with parallel streams.
- Overusing parallel streams on small collections, where the overhead outweighs the benefit.
