# Transfer Web API

## Quick Start
```bash
mvn spring-boot:run           # Dev server on http://localhost:8080
mvn test                       # Run all tests
mvn test -Dtest=TransferServiceTest  # Run specific test class
```

## Tech Stack
- Java 25 (README says 21, but pom.xml and CI use 25)
- Spring Boot 4.0.1
- Maven (wrapper: `./mvnw`)
- H2 in-memory database

## Profiles
| Profile | DB Password | H2 Console | Logging | DDL Auto |
|---------|-------------|------------|---------|----------|
| `dev`   | (empty)     | enabled    | INFO + SQL | create-drop |
| default | `password`  | disabled   | WARN | N/A |

Run dev: `mvn spring-boot:run -Pdev`
CI build: `mvn -B package -Pdev`

Dev profile shows SQL queries and DDL statements; default profile is for production with minimal logging.

## Architecture
- **Concurrency**: `@Version` on `Customer` entity for optimistic locking
- **History**: `TransferLogService` uses `REQUIRES_NEW` propagation - failed transfers are always logged
- **Limit**: R$10,000.00 per transfer

## Package Structure
```
src/main/java/com/itau/transferencia/
├── controllers/   # REST endpoints
├── services/     # Business logic
├── repositories/ # JPA
├── entities/     # JPA entities
├── dtos/         # Request/Response
├── exceptions/   # Custom exceptions
├── validations/  # Bean Validation
└── config/       # Spring config
```

## Key Endpoints
- `/v1/customers` - API endpoints
- `/swagger-ui/index.html` - Interactive API docs
- `/api-docs` - OpenAPI JSON
- `/actuator/health` - Health check

## Error Handling Pattern
- **CustomException Hierarchy**: All business errors extend `BusinessException(message, HttpStatus)` which carries HTTP status codes
  - Examples: `AccountNotFoundException`, `InsufficientFundsException`, `SameAccountException`
- **Global Exception Mapping** via `@RestControllerAdvice`:
  - `BusinessException` → Custom HTTP status from exception
  - `ObjectOptimisticLockingFailureException` (concurrency) → 400 Bad Request
  - `DataIntegrityViolationException` (unique constraint) → 409 Conflict
  - `MethodArgumentNotValidException` (validation) → 400 with field errors
- Error responses use `ErrorDTO` with message/error list
- Failed transfers trigger audit via `TransferLogService.save()` before exception is thrown

## Validation Pattern
- Custom `@AccountNumber` annotation (`validations/` package) validates format `xxxxx-x` (5 digits, hyphen, 1 digit)
- Transferred amount uses `@Min(1)` and `@Max(10_000)` constraints in request DTOs (records)
- Bean Validation (`jakarta.validation`) integrated with controller parameter binding
- Errors collected and returned as list in `ErrorDTO`

## Entity & DTO Patterns
- **Entities** (`entities/` package): JPA entities with:
  - `@Version` field on `Customer` for optimistic locking
  - `@PrePersist` lifecycle callbacks (Transfer auto-sets `createdAt`)
  - Static factory methods: `Transfer.ofCompleted()`, `Transfer.ofFailed()`
  - Helper methods: `Customer.canTransfer(amount)`, `Customer.isSameAccount()`
- **DTOs** (separate `request/`, `response/` packages): Java records for type-safe serialization
  - Request: `CustomerDTO`, `TransferDTO` with validation annotations
  - Response: `CustomerDTO`, `TransferDTO`, `ErrorDTO`

## Transaction Management
- `transfer()` method marked `@Transactional` in `TransferServiceImpl`
  - Throws `BusinessException` or `ObjectOptimisticLockingFailureException` on failure
- `TransferLogService.save()` uses `@Transactional(propagation = REQUIRES_NEW)`
  - Ensures audit records persist independently even if parent transaction rolls back
  - Guarantees history for failed transfers for compliance

## Testing
- **Test Framework**: JUnit 5 with Mockito (`@ExtendWith(MockitoExtension.class)`)
- **Unit Test Pattern**: Mock repository and dependencies, test service logic in isolation
  - Verify business rule enforcement (balance checks, account validation, audit calls)
  - Use static factory methods on entities for test data
- **Location**: `src/test/java/com/itau/transferencia/services/`
- **Run specific test**: `mvn test -Dtest=TransferServiceTest`

## Notes
- ControllerAdvice handles global exception mapping
- OpenAPI spec generated to `target/docs/openapi.json` during `integration-test` phase
- API versioning via `/v1/` prefix
- Spring DevTools included for rapid local development
- JSON serialization uses `@JsonBackReference`/`@JsonManagedReference` to prevent circular references
