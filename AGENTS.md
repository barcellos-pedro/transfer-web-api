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
| Profile | DB Password | H2 Console | Logging |
|---------|-------------|------------|---------|
| `dev`   | (empty)     | enabled    | INFO + SQL |
| default | `password`  | disabled   | WARN |

Run dev: `mvn spring-boot:run -Pdev`
CI build: `mvn -B package -Pdev`

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

## Notes
- ControllerAdvice handles global exception mapping
- OpenAPI spec generated to `target/docs/openapi.json` during `integration-test` phase