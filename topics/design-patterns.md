# Common Design Patterns in Java

Idiomatic Java implementations of the classic Gang-of-Four patterns most used in real codebases.

## Creational

### Singleton
Ensures a class has exactly one instance. Prefer the enum form — it's thread-safe and serialization-safe for free.

```java
public enum ConfigManager {
    INSTANCE;
    private final Map<String, String> settings = new HashMap<>();
    public String get(String key) { return settings.get(key); }
}
```

### Builder
Constructs complex objects step by step, avoiding telescoping constructors.

```java
public class Pizza {
    private final String size;
    private final List<String> toppings;

    private Pizza(Builder b) { this.size = b.size; this.toppings = b.toppings; }

    public static class Builder {
        private String size = "medium";
        private List<String> toppings = new ArrayList<>();

        public Builder size(String size) { this.size = size; return this; }
        public Builder topping(String t) { toppings.add(t); return this; }
        public Pizza build() { return new Pizza(this); }
    }
}

Pizza pizza = new Pizza.Builder().size("large").topping("cheese").build();
```

### Factory Method
Delegates object creation to subclasses or a dedicated method instead of calling `new` directly.

```java
public interface Notifier { void send(String msg); }

public class NotifierFactory {
    public static Notifier create(String type) {
        return switch (type) {
            case "email" -> new EmailNotifier();
            case "sms" -> new SmsNotifier();
            default -> throw new IllegalArgumentException("unknown type: " + type);
        };
    }
}
```

## Structural

### Adapter
Converts one interface into another the client expects.

```java
public class XmlToJsonAdapter implements JsonSource {
    private final XmlSource xmlSource;
    public XmlToJsonAdapter(XmlSource xmlSource) { this.xmlSource = xmlSource; }
    public String getJson() { return convert(xmlSource.getXml()); }
}
```

### Decorator
Adds behavior to an object dynamically by wrapping it, without changing its class. `java.io` streams are the canonical example (`BufferedReader` wrapping a `Reader`).

```java
Reader reader = new BufferedReader(new FileReader("data.txt"));
```

### Facade
Provides a simplified interface over a complex subsystem, hiding its internals from callers.

## Behavioral

### Strategy
Encapsulates interchangeable algorithms behind a common interface, selected at runtime. In modern Java this is often just a lambda implementing a functional interface.

```java
Comparator<Person> byAge = (a, b) -> a.getAge() - b.getAge();
Comparator<Person> byName = Comparator.comparing(Person::getName);
people.sort(byAge); // strategy swapped at call time
```

### Observer
Lets subscribers react to events published by a subject, without tight coupling. Java's `PropertyChangeListener` is a built-in example; reactive libraries (RxJava, Project Reactor) generalize the idea.

```java
public interface OrderListener { void onOrderPlaced(Order order); }

public class OrderService {
    private final List<OrderListener> listeners = new ArrayList<>();
    public void subscribe(OrderListener l) { listeners.add(l); }
    public void placeOrder(Order order) {
        listeners.forEach(l -> l.onOrderPlaced(order));
    }
}
```

### Template Method
Defines the skeleton of an algorithm in a base class, letting subclasses override specific steps.

```java
public abstract class DataProcessor {
    public final void process() {
        load();
        transform();
        save();
    }
    protected abstract void load();
    protected abstract void transform();
    protected void save() { System.out.println("saved"); } // default step
}
```

## When to Reach for a Pattern

Patterns are named solutions to recurring problems, not a checklist to apply everywhere. Modern Java (lambdas, records, sealed types) often replaces what used to require Strategy, Visitor, or Builder boilerplate — introduce a named pattern when it genuinely clarifies intent or removes duplication, not by default.
