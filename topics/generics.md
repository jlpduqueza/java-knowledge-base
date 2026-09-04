# Generics in Java

Generics let types (classes, interfaces, methods) be parameterized over other types, giving compile-time type safety without casting.

## Why Generics

Before generics, collections stored `Object` and required casts:

```java
List list = new ArrayList();
list.add("hello");
String s = (String) list.get(0); // manual cast, unsafe
```

With generics, the compiler enforces type correctness:

```java
List<String> list = new ArrayList<>();
list.add("hello");
String s = list.get(0); // no cast needed
```

## Generic Classes

```java
public class Box<T> {
    private T value;

    public void set(T value) { this.value = value; }
    public T get() { return value; }
}

Box<Integer> intBox = new Box<>();
intBox.set(42);
```

## Generic Methods

```java
public static <T> T firstElement(List<T> list) {
    return list.get(0);
}
```

## Bounded Type Parameters

Restrict a type parameter to a subtype of a given type:

```java
public static <T extends Comparable<T>> T max(List<T> list) {
    T result = list.get(0);
    for (T item : list) {
        if (item.compareTo(result) > 0) result = item;
    }
    return result;
}
```

## Wildcards

- `List<?>` — unknown type ("unbounded wildcard").
- `List<? extends Number>` — producer; you can read `Number`s out, but cannot add (except `null`).
- `List<? super Integer>` — consumer; you can add `Integer`s in, but reads come out as `Object`.

The mnemonic is **PECS**: Producer `extends`, Consumer `super`.

```java
public static double sum(List<? extends Number> numbers) {
    double total = 0;
    for (Number n : numbers) total += n.doubleValue();
    return total;
}
```

## Type Erasure

Java implements generics via erasure: generic type information exists only at compile time and is removed (erased to `Object` or the bound) in the compiled bytecode. Consequences:

- You cannot do `new T()` or `new T[]` directly.
- You cannot use `instanceof` with a parameterized type (`obj instanceof List<String>` is illegal; `obj instanceof List<?>` is fine).
- Overloads that differ only by generic type parameter are not allowed — they erase to the same signature.

## Generic Interfaces

```java
public interface Repository<T, ID> {
    T findById(ID id);
    void save(T entity);
}

public class UserRepository implements Repository<User, Long> {
    public User findById(Long id) { ... }
    public void save(User entity) { ... }
}
```
