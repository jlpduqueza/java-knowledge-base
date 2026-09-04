# Records, Sealed Types, and Pattern Matching (Modern Java)

Features from Java 14-21 that make data-centric and conditional code far more concise and safer.

## Records (Java 16+)

A record is a concise, immutable data carrier. The compiler generates a canonical constructor, private final fields, accessors, `equals()`, `hashCode()`, and `toString()`.

```java
public record Point(int x, int y) {}

Point p = new Point(3, 4);
p.x();          // accessor, not getX()
p.toString();   // "Point[x=3, y=4]"
p.equals(new Point(3, 4)); // true - value-based equality
```

### Compact Constructors
Add validation without restating all the fields:

```java
public record Range(int min, int max) {
    public Range {
        if (min > max) throw new IllegalArgumentException("min > max");
    }
}
```

### Records Can Implement Interfaces and Have Extra Methods

```java
public record Money(long cents, String currency) implements Comparable<Money> {
    public double toDollars() { return cents / 100.0; }

    @Override
    public int compareTo(Money other) { return Long.compare(cents, other.cents); }
}
```

Records cannot extend another class (they implicitly extend `Record`) and their fields are always `final` — use them for immutable data, not for entities with identity or mutable state.

## Sealed Classes and Interfaces (Java 17+)

Restrict which classes are allowed to extend/implement a type, giving exhaustiveness guarantees the compiler can check.

```java
public sealed interface Shape permits Circle, Square, Triangle {}

public record Circle(double radius) implements Shape {}
public record Square(double side) implements Shape {}
public record Triangle(double base, double height) implements Shape {}
```

## Pattern Matching for instanceof (Java 16+)

```java
if (obj instanceof String s && !s.isEmpty()) {
    System.out.println(s.toUpperCase()); // s already cast, no manual cast needed
}
```

## Switch Expressions (Java 14+)

```java
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY -> 7;
    default -> {
        int len = day.toString().length();
        yield len; // yield returns a value from a block branch
    }
};
```

## Pattern Matching for switch (Java 21+)

Combines sealed types + pattern matching for exhaustive, type-safe dispatch without a chain of `instanceof` checks:

```java
double area = switch (shape) {
    case Circle c -> Math.PI * c.radius() * c.radius();
    case Square s -> s.side() * s.side();
    case Triangle t -> 0.5 * t.base() * t.height();
    // no default needed - the compiler knows these are the only permitted subtypes
};
```

### Record Patterns (Java 21+)
Destructure a record directly in the pattern:

```java
record Point(int x, int y) {}

static String describe(Object obj) {
    return switch (obj) {
        case Point(int x, int y) when x == y -> "on the diagonal";
        case Point(int x, int y) -> "(" + x + ", " + y + ")";
        default -> "not a point";
    };
}
```

## Why This Matters

Together these features push Java toward expressing "what data looks like" and "what to do for each shape of data" directly, replacing a lot of boilerplate (getters/equals/hashCode, instanceof-and-cast chains, non-exhaustive if/else ladders) that used to require manual code or Lombok-style code generation.
