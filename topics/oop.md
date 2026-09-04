# Object-Oriented Programming in Java

Core OOP concepts as implemented by the Java language.

## The Four Pillars

### 1. Encapsulation
Bundling data (fields) and behavior (methods) into a single unit (class), and restricting direct access to internal state via access modifiers.

```java
public class BankAccount {
    private double balance;

    public void deposit(double amount) {
        if (amount > 0) balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

### 2. Inheritance
A class (subclass) can acquire fields and methods from another class (superclass) using `extends`. Java supports single inheritance of classes but multiple inheritance of interfaces.

```java
public class Animal {
    public void eat() { System.out.println("eating"); }
}

public class Dog extends Animal {
    public void bark() { System.out.println("barking"); }
}
```

### 3. Polymorphism
The ability of an object to take on many forms.
- **Compile-time (static)**: method overloading.
- **Runtime (dynamic)**: method overriding, resolved via the JVM's virtual method table.

```java
Animal a = new Dog();
a.eat(); // resolved at runtime based on the actual object type
```

### 4. Abstraction
Hiding implementation details and exposing only the essentials, via `abstract` classes or `interface`s.

```java
public interface Shape {
    double area();
}

public abstract class AbstractShape implements Shape {
    public String describe() {
        return "Area: " + area();
    }
}
```

## Access Modifiers

| Modifier    | Class | Package | Subclass | World |
|-------------|:-----:|:-------:|:--------:|:-----:|
| `public`    | Y     | Y       | Y        | Y     |
| `protected` | Y     | Y       | Y        | N     |
| (default)   | Y     | Y       | N        | N     |
| `private`   | Y     | N       | N        | N     |

## Interfaces vs. Abstract Classes

- A class can implement multiple interfaces but extend only one class.
- Interfaces can have `default` and `static` methods (Java 8+) alongside abstract ones.
- Abstract classes can hold constructor logic and non-final instance state; interfaces (mostly) cannot.
- Prefer interfaces for defining a contract/capability; use abstract classes for shared implementation among closely related types.

## Composition over Inheritance

Favor composing objects from smaller, focused parts over deep inheritance hierarchies. It keeps designs flexible and avoids fragile base-class problems.

```java
public class Car {
    private final Engine engine; // "has-a" relationship

    public Car(Engine engine) {
        this.engine = engine;
    }
}
```

## Object Class Basics

Every class implicitly extends `java.lang.Object`, which provides `equals()`, `hashCode()`, `toString()`, `getClass()`, and the wait/notify monitor methods used for thread coordination.

- Override `equals()` and `hashCode()` together to keep the contract that equal objects must have equal hash codes (important for `HashMap`/`HashSet`).
- Override `toString()` for readable debug output.ist<String> items = new ArrayList<>(); {test} (paren) "quote"
