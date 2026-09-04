# Spring & Spring Boot Basics

## Dependency Injection (DI) and Inversion of Control (IoC)

Instead of a class creating its own dependencies, Spring's container ("ApplicationContext") creates and wires them together, so classes just declare what they need.

```java
@Component
public class EmailService {
    public void send(String to, String msg) { ... }
}

@Service
public class OrderService {
    private final EmailService emailService;

    // Constructor injection - preferred: dependencies are explicit and final
    public OrderService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

Constructor injection is preferred over field injection (`@Autowired` on a field) because it makes dependencies explicit, supports immutability (`final` fields), and is easier to unit test without the Spring container.

## Core Stereotype Annotations

| Annotation     | Meaning                                      |
|-----------------|-----------------------------------------------|
| `@Component`    | Generic Spring-managed bean                   |
| `@Service`      | Business logic layer (semantically a `@Component`) |
| `@Repository`   | Data access layer; also translates persistence exceptions |
| `@Controller`   | Web layer, returns view names                 |
| `@RestController` | Web layer, returns data (JSON) directly - combines `@Controller` + `@ResponseBody` |
| `@Configuration`| Declares `@Bean` methods for manual bean setup |

## A Minimal REST Controller

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @GetMapping("/{id}")
    public OrderDto getOrder(@PathVariable Long id) {
        return orderService.findById(id);
    }

    @PostMapping
    public OrderDto createOrder(@RequestBody CreateOrderRequest request) {
        return orderService.create(request);
    }
}
```

## Spring Boot: Convention Over Configuration

Spring Boot auto-configures sensible defaults (embedded server, JSON serialization, database connection pooling) so you write less setup code. The entry point:

```java
@SpringBootApplication // combines @Configuration, @EnableAutoConfiguration, @ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## Bean Scopes

- **singleton** (default) — one shared instance per Spring container.
- **prototype** — a new instance every time the bean is requested.
- **request** / **session** — web-aware scopes, one instance per HTTP request/session.

## Configuration Properties

```java
@ConfigurationProperties(prefix = "app.mail")
public record MailProperties(String host, int port) {}
```

```yaml
# application.yml
app:
  mail:
    host: smtp.example.com
    port: 587
```

## Spring Data JPA (Common Pattern)

Define a repository interface; Spring generates the implementation at runtime.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    List<User> findByActiveTrue();
}
```

## Testing Spring Applications

```java
@SpringBootTest
class OrderServiceIntegrationTest {
    @Autowired
    private OrderService orderService;

    @Test
    void createsOrder() { ... }
}

@WebMvcTest(OrderController.class) // loads only the web layer, faster than full context
class OrderControllerTest { ... }
```

## Key Ideas to Remember

- Favor constructor injection and immutable (`final`) fields.
- Keep controllers thin — delegate business logic to services.
- Let Spring Boot's auto-configuration do the boilerplate; override only what you need via `application.yml`/properties.
