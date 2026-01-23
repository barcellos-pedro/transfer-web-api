# Financial Transfers API

This project consists of a RESTful API for customer management and financial transfers.

## Technologies and Requirements

* **Java 25**
* **Spring Boot 4**
* **H2**
* **Maven**

## How to Run the Application

1. **Prerequisites**: Make sure you have **JDK 21** and **Maven** installed.
2. **Clone the repository**:

```bash
git clone https://github.com/barcellos-pedro/api-transferencia.git
cd api-transferencia
```

1. **Compile and run**:

```bash
mvn spring-boot:run
```

1. **Access the API**: The application will be available at `http://localhost:8080`.
2. **Interactive Documentation (Swagger)**: Access `http://localhost:8080/swagger-ui/index.html` to test the endpoints
   in the browser

## Main Endpoints

The API follows RESTful patterns and URL versioning (`/v1/...`):

**Customers**:

* `GET /v1/customers`: General customer listing.
* `POST /v1/customers`: Register new customer.
* `GET /v1/customers/{account}`: Search Customer by account number.
* `GET /v1/customers/{account}/transfers`: History ordered by descending date, including failures.
* `POST /v1/customers/{account}/transfers`: Perform transfer between accounts (Limit of R$ 10,000.00).

## Additional Endpoints

* `GET /api-docs` - OpenAPI specification in JSON.
* `GET /actuator/health`: Application health check.
* `GET /swagger-ui/index.html`: Visual interface to test API endpoints.

## Monitoring and Documentation

The API uses Spring Boot Actuator to provide metrics and health status, essential for production environments and
observability.

**Diagnostic Endpoints (Actuator)**

| Endpoint              | Description        | Purpose                                                 |
|-----------------------|--------------------|---------------------------------------------------------|
| GET /actuator/health  | Application Health | Checks if App, DB and Disk are operational.             |
| GET /actuator/metrics | Metrics            | Lists available metrics (JVM, CPU, HTTP Requests).      |
| GET /actuator/info    | Information        | Custom data about project version and build.            |

## Engineering & Architecture Decisions

### 1. History Resilience

As requested, unsuccessful transfers are also stored. To ensure that the failure record is
persisted even when the financial transaction suffers rollback, I used **`REQUIRES_NEW`** propagation in the audit
service. This guarantees the integrity of the history for banking compliance.

### 2. Concurrency Control

To meet the concurrency control requirement in the transfer operation, the following was implemented:

* **Pessimistic Locking** (or **Optimistic Locking** with `@Version`): To avoid the "Lost Update" problem when
  two processes try to debit from the same account simultaneously.

### 3. Business Rules Validation

The rules of sufficient balance and maximum limit of R$ 10,000.00 per operation were centralized in the service layer,
ensuring that the database state remains consistent.

## Tests

Test coverage was prioritized to ensure the reliability of transfers:

* **Unit Tests**: Validation of business logic and balance calculations.
* **Integration Tests**: Complete transfer flow simulating concurrency and database rollback.

Run the tests with:

```bash
mvn test
```

## Project Base

The project was structured using [Spring Initializr](https://start.spring.io).

**Technology Stack and Dependencies**

Below, the main components selected to meet the API requirements:

<img width="1710" height="654" alt="image" src="https://github.com/user-attachments/assets/35a7e8da-9396-4ad3-944b-87a0fa95573a" />
