---
description: Back coding conventions for ab-java-react
globs: "back/src/**/*.java"
---
# Back Conventions — ab-java-react

## Summary

Java 21 + Spring Boot REST tier. Constructor-injected, stateless `@Service` beans orchestrate immutable domain records and Spring Data JPA repositories behind thin `@RestController`s. Conventions follow the Google Java Style Guide and idiomatic Spring Boot (favor immutability, `record` DTOs, and package-by-feature).

---

## Naming

### Folders
- **Pattern**: hybrid — package-by-feature, layered within
- **Casing**: lowercase (Java package convention)
- **Examples**: `health`, `shared`

### Files
- **Entity**: `{Noun}.java` — `HealthCheck.java`
- **DTO**: `{Noun}Response.java` / `{Noun}Request.java` — `HealthResponse.java`
- **Repository**: `{Noun}Repository.java` — `HealthCheckRepository.java`
- **Service**: `{Noun}Service.java` — `HealthService.java`
- **Controller**: `{Noun}Controller.java` — `HealthController.java`

### Language Elements
- **Classes / Records**: PascalCase — `HealthService`
- **Interfaces / Types**: PascalCase (no `I` prefix) — `HealthCheckRepository`
- **Methods / Functions**: camelCase — `checkHealth`
- **Variables / Fields**: camelCase — `checkedAt`
- **Constants**: UPPER_SNAKE — `MAX_RETRIES`
- **Enums / Enum members**: PascalCase / UPPER_SNAKE — `Status.UP`

---

## Artifact Roles

### Entity

**Dominant pattern**: JPA `@Entity` class with a generated id; no business logic, persistence-mapped fields only.

**Canonical example**:
```java
@Entity
@Table(name = "health_check")
public class HealthCheck {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  @Column(nullable = false) private String status;
  @Column(name = "database_status", nullable = false) private String databaseStatus;
  @Column(name = "checked_at", nullable = false) private Instant checkedAt;
  protected HealthCheck() {} // JPA
  public HealthCheck(String status, String databaseStatus, Instant checkedAt) { /* assign */ }
  // getters only — instances are effectively immutable after creation
}
```

**Anti-pattern**:
- Exposing setters or leaking entities across the controller boundary — map to a DTO instead.

### DTO

**Dominant pattern**: immutable Java `record` carrying data across the API boundary; no framework annotations beyond validation.

**Canonical example**:
```java
public record HealthResponse(String status, String database, Instant checkedAt) {}
```

**Anti-pattern**:
- Returning the JPA entity directly from a controller (couples wire format to schema).

### Repository

**Dominant pattern**: interface extending `JpaRepository`; derive queries from method names, no `@Query` unless necessary.

**Canonical example**:
```java
public interface HealthCheckRepository extends JpaRepository<HealthCheck, Long> {
  Optional<HealthCheck> findFirstByOrderByCheckedAtDesc();
}
```

**Anti-pattern**:
- Putting business rules in repositories — they only persist and query.

### Service

**Dominant pattern**: `@Service` with constructor injection; holds business logic, returns DTOs/domain, never `ResponseEntity`.

**Canonical example**:
```java
@Service
public class HealthService {
  private final HealthCheckRepository repository;
  public HealthService(HealthCheckRepository repository) { this.repository = repository; }

  public HealthResponse check() {
    String db = probeDatabase() ? "UP" : "DOWN";
    String status = "UP".equals(db) ? "UP" : "DOWN";
    HealthCheck saved = repository.save(new HealthCheck(status, db, Instant.now()));
    return new HealthResponse(saved.getStatus(), saved.getDatabaseStatus(), saved.getCheckedAt());
  }
}
```

**Anti-pattern**:
- Field injection (`@Autowired` on fields) — use constructor injection for testability and immutability.

### Controller

**Dominant pattern**: thin `@RestController` mapping HTTP to a service call; no business logic.

**Canonical example**:
```java
@RestController
@RequestMapping("/api/health")
public class HealthController {
  private final HealthService service;
  public HealthController(HealthService service) { this.service = service; }

  @GetMapping
  public HealthResponse health() { return service.check(); }
}
```

**Anti-pattern**:
- Accessing repositories directly from controllers — always go through a service.

---

## Wiring and Dependencies

Constructor injection only (no field/setter injection). Features reference shared config under `shared/`. See `back.arch.md` — Components / Dependencies between modules.

## Error Handling

Use a single `@RestControllerAdvice` to map exceptions to RFC 7807 `ProblemDetail`. Throw domain exceptions from services; never return raw stack traces. Validate inputs with Bean Validation (`@Valid`).

---

## Testing

### Infrastructure
- **Framework**: JUnit 5 + Spring Boot Test + MockMvc; Mockito for unit isolation
- **Runner**: `./mvnw test`
- **Placement**: mirrored structure under `src/test/java`
- **File naming**: `*Test.java` (unit), `*IT.java` (integration)

### Patterns
- **Setup**: `@WebMvcTest` for controllers, plain Mockito for services, `@DataJpaTest` for repositories
- **Mocking**: mock collaborators with `@MockBean`/`@Mock`; never mock the class under test
- **Assertions**: AssertJ (`assertThat(...)`)

### Canonical Test Example

```java
@WebMvcTest(HealthController.class)
class HealthControllerTest {
  @Autowired MockMvc mvc;
  @MockBean HealthService service;

  @Test
  void returnsUpStatus() throws Exception {
    given(service.check()).willReturn(new HealthResponse("UP", "UP", Instant.EPOCH));
    mvc.perform(get("/api/health"))
       .andExpect(status().isOk())
       .andExpect(jsonPath("$.status").value("UP"));
  }
}
```

### Coverage by Artifact Role

| Role | Tested | Scope |
|------|--------|-------|
| Controller | Yes | happy path + error mapping |
| Service | Yes | happy path + DB-down case |
| Repository | Yes | query methods |
| DTO / Entity | No | no behavior to test |

### What NOT to Test
- Framework wiring, getters, and Spring Data derived queries' internals — trust the framework.

---

## Known Deviations

- No deviations yet — greenfield baseline.
