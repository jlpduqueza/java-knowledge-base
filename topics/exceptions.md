# Exception Handling in Java

## The Exception Hierarchy

```
Throwable
 |- Error                    (unrecoverable: OutOfMemoryError, StackOverflowError)
 |- Exception
     |- RuntimeException      (unchecked: NullPointerException, IllegalArgumentException...)
     |- (everything else)     (checked: IOException, SQLException...)
```

## Checked vs. Unchecked

- **Checked exceptions** (`Exception` but not `RuntimeException`) must be either caught or declared with `throws`. The compiler enforces handling. Used for recoverable conditions the caller should anticipate (e.g. a missing file).
- **Unchecked exceptions** (`RuntimeException` and its subclasses) are not required to be declared or caught. Typically signal programming errors (null dereference, bad argument, illegal state).
- **Errors** (`Error` subclasses) indicate serious problems an application usually should not try to catch.

## Basic Syntax

```java
try {
    riskyOperation();
} catch (IOException e) {
    log.error("IO failed", e);
} catch (SQLException e) {
    log.error("DB failed", e);
} finally {
    cleanup(); // always runs, even if try/catch returns or throws
}
```

## Try-With-Resources

Automatically closes any `AutoCloseable` resource, even on exception, without a manual `finally` block:

```java
try (BufferedReader reader = new BufferedReader(new FileReader("data.txt"))) {
    return reader.readLine();
} catch (IOException e) {
    throw new UncheckedIOException(e);
}
```

## Custom Exceptions

```java
public class InsufficientFundsException extends RuntimeException {
    public InsufficientFundsException(String message) {
        super(message);
    }
}
```

Guidelines:
- Extend `RuntimeException` unless callers genuinely need to be forced to handle the failure.
- Always preserve the original cause: `throw new ServiceException("failed", originalException);`
- Don't use exceptions for ordinary control flow — they're relatively expensive and hurt readability.

## Multi-Catch and Rethrow

```java
try {
    process();
} catch (IOException | SQLException e) {
    log.error("processing failed", e);
    throw e; // precise rethrow: compiler knows it's IOException or SQLException
}
```

## Common Pitfalls

- Swallowing exceptions silently (`catch (Exception e) {}`) hides real bugs.
- Catching `Exception` or `Throwable` broadly instead of the specific type you can actually handle.
- Losing the original stack trace by wrapping without passing the cause.
- Using exceptions to signal expected, frequent outcomes (e.g. "not found") instead of returning `Optional` or a result type.
