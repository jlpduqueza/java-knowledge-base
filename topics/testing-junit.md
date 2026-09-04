# Testing Java with JUnit and Mockito

## JUnit 5 Basics

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {

    @Test
    void addsTwoNumbers() {
        Calculator calc = new Calculator();
        assertEquals(5, calc.add(2, 3));
    }

    @Test
    void throwsOnDivideByZero() {
        Calculator calc = new Calculator();
        assertThrows(ArithmeticException.class, () -> calc.divide(1, 0));
    }
}
```

## Lifecycle Annotations

| Annotation      | When it runs                          |
|------------------|-----------------------------------------|
| `@BeforeAll`     | Once, before all tests in the class (must be `static`) |
| `@BeforeEach`    | Before every test method                |
| `@Test`          | Marks a test method                     |
| `@AfterEach`     | After every test method                 |
| `@AfterAll`      | Once, after all tests (must be `static`)|
| `@Disabled`      | Skip this test                          |

```java
class OrderServiceTest {
    private OrderService service;

    @BeforeEach
    void setUp() { service = new OrderService(); }

    @Test
    void placesOrderSuccessfully() { ... }
}
```

## Parameterized Tests

```java
@ParameterizedTest
@ValueSource(ints = {1, 2, 3, 5, 8})
void allInputsAreLessThanTen(int input) {
    assertTrue(input < 10);
}

@ParameterizedTest
@CsvSource({"1,1,2", "2,3,5", "10,-5,5"})
void addsCorrectly(int a, int b, int expected) {
    assertEquals(expected, new Calculator().add(a, b));
}
```

## Assertions

```java
assertEquals(expected, actual);
assertTrue(condition);
assertNotNull(object);
assertThrows(SomeException.class, () -> riskyCall());
assertAll(
    () -> assertEquals(1, x),
    () -> assertEquals(2, y)
); // runs all assertions even if one fails, reports all failures together
```

## Mockito: Mocking Dependencies

Isolate the unit under test by replacing its collaborators with test doubles.

```java
import static org.mockito.Mockito.*;

class OrderServiceTest {

    @Test
    void sendsConfirmationEmailAfterOrder() {
        EmailClient emailClient = mock(EmailClient.class);
        OrderService service = new OrderService(emailClient);

        service.placeOrder(new Order("A123"));

        verify(emailClient).send(anyString(), contains("A123"));
    }

    @Test
    void returnsFallbackWhenRepositoryFails() {
        UserRepository repo = mock(UserRepository.class);
        when(repo.findById(1L)).thenThrow(new RuntimeException("db down"));

        UserService service = new UserService(repo);

        assertEquals(User.GUEST, service.getUserOrGuest(1L));
    }
}
```

Key Mockito verbs: `mock()` creates a fake, `when(...).thenReturn(...)` stubs behavior, `verify(...)` asserts an interaction happened, `@Mock`/`@InjectMocks` (with `@ExtendWith(MockitoExtension.class)`) wire things up with less boilerplate.

## Test Structure: Arrange-Act-Assert

```java
@Test
void test() {
    // Arrange - set up inputs and dependencies
    var cart = new ShoppingCart();
    cart.add(new Item("book", 10.0));

    // Act - call the thing being tested
    double total = cart.getTotal();

    // Assert - check the outcome
    assertEquals(10.0, total);
}
```

## Testing Guidelines

- Test behavior, not implementation details — refactoring shouldn't break tests unless behavior changed.
- One logical assertion focus per test; name tests to describe the scenario (`returnsEmptyList_whenNoResultsFound`).
- Prefer real objects for simple, fast collaborators; mock only what's slow, external, or non-deterministic (databases, network, time).
- Keep tests independent — no shared mutable state or ordering assumptions between tests.
