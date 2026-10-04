---
name: unit-tests
description: >
  Generates full-flow unit tests from a controller endpoint,
  traversing controller, service, and helper functions, mocking ALL
  external dependencies (database, HTTP, queues, cache, third-party services). Covers
  happy paths and edge cases, maximizing coverage throughout the flow. Use when
  the user provides an endpoint (route + HTTP method) or the controller file
  and requests unit tests.
argument-hint: "Provide the endpoint (e.g.: POST /api/v1/users) or the controller path"
---

# Flow-Based Unit Test Generator

## Objective

Generate unit tests that cover the **full flow from the controller endpoint**,
traversing all involved layers (controller → service → helpers/utils), **without touching
the real database or any external dependency**. Every interaction with database,
HTTP, queue, cache, or third-party service must be **mocked**.

The focus is to **maximize line and branch coverage throughout the flow**, covering:
- happy path
- edge cases (limits, empty values, nulls, invalid formats, duplication, concurrency)

## When to use

Use this skill when the user:
- Provides an **endpoint** (e.g.: `POST /api/v1/users`) and asks for tests
- Provides the **controller file** and asks for flow unit tests
- Requests unit tests with database **mocks** for an entire flow

**DO NOT use** when the user wants:
- E2E or integration tests with a real database
- Unit tests for an isolated function without flow context
- UI tests

## Execution process

### Step 1 — Map the flow from the controller

1. Locate the controller file and the endpoint method.
2. Identify the **entry point**: route, HTTP method, input DTO, output DTO.
3. Trace the call to the corresponding **service**.
4. Inside the service, identify:
   - Called repositories/DAOs (database)
   - External HTTP calls
   - Queues, cache, storage
   - Invoked helper functions and utilities
5. Repeat tracing recursively until **all I/O calls** are mapped.
6. Produce a **textual flow diagram** before writing any tests:
```
POST /api/v1/users
└─ UserController.create(dto)
└─ UserService.create(dto)
├─ UserValidator.validate(dto) [pure — test directly]
├─ UserRepository.findByEmail(email) [DATABASE — mock]
├─ HashService.hash(password) [pure — test directly]
└─ UserRepository.save(user) [DATABASE — mock]
```
7. **Present the map to the user and wait for confirmation** before generating tests.
If something cannot be traced (missing file, unknown layer), list it under
"UNMAPPED POINTS" and request clarification.

### Step 2 — Identify dependencies to mock

Mandatory **mocking**:
- Repositories, DAOs, ORMs (JPA, TypeORM, Prisma, Eloquent, GORM, etc.)
- HTTP clients (Feign, Axios, HttpClient, RestTemplate)
- Queues (Kafka, RabbitMQ, SQS)
- Cache (Redis, Memcached)
- Storage (S3, GCS)
- Third-party services (email, SMS, payments)
- Clock/current date (when it affects the flow)

**DO NOT mock** (must be tested for real):
- Pure validation functions
- Formatting helpers
- Business rules in the service
- Mappers/DTO converters
- Internal calculations

### Step 3 — Generate tests

For each flow layer:

**Controller** (input test):
- Test DTO serialization/deserialization
- Test input validation (required fields, formats)
- Test HTTP response mapping (200, 201, 400, 404, 409, 422, 500)
- Test propagation of service exceptions
- Verify that the service was called with the correct parameters

**Service** (business rule test):
- Test each decision branch (if/else, switch, try/catch)
- Test correct calls to repositories (with expected parameters)
- Test handling of external dependency exceptions
- Test idempotency when applicable
- Test limits (empty, null, 0, negative, maximum size)

**Helpers/utils** (direct test):
- Test pure functions with valid and invalid inputs
- Cover limits and extreme formats

### Step 4 — Mandatory coverage

For each flow, ensure cases for:

| Category | Examples |
|---|---|
| **Happy path** | Valid input → expected success |
| **Invalid input** | Missing required field, wrong format, wrong type |
| **Limits** | Empty, 0, negative, max/min length, null, undefined |
| **Business states** | Duplication, not found, already exists, expired |
| **Dependency errors** | Database throws exception, external HTTP timeout, queue unavailable |
| **Concurrency** (when applicable) | Two simultaneous calls, race condition |
| **Authorization** | No permission, invalid token, inactive user |

## Output format

### 1. Flow Map
Textual diagram (see Step 1) with all layers and identified dependencies.

### 2. Dependencies to Mock
Explicit list of what will be mocked and why.

### 3. Test Cases (CT format)
For each case:

TU-NNN — [Short and direct title]
Layer: [Controller | Service | Helper]
Scenario: [happy | edge | error]
Pre: [required mocks and state]
Action: [executed call]
Checks:

[assertion 1]

[assertion 2]


### 4. Test Code
Generate tests in the **project's test framework** (automatically identify:
JUnit/Mockito for Java, Jest for Node, pytest for Python, etc.).
Follow the AAA pattern (Arrange, Act, Assert).
Name tests according to the project's naming standard.

### 5. Expected Coverage
Table: lines/branches covered by layer, and what was left out.

### 6. UNMAPPED POINTS
List of flow sections that could not be traced and what is missing to cover them.

### 7. Coverage Execution Report
At the end of test generation, execute the tests and return the coverage report for the tested classes.
If there is any tooling blocker for coverage calculation (missing binary, plugin, reporter, or config),
return a request asking for installation of the required tool before proceeding.

## Generation rules

- **Never** access a real database. If the code calls the database, mock it.
- **Always** verify whether the mock was called with the correct parameters.
- **Always** test the exception path, not only the success path.
- **Do not** generate trivial tests (getters, setters, empty constructors).
- **Do not** test the framework (e.g., testing whether Spring injects dependencies).
- **Do not** duplicate tests across layers — if already covered in service, do not repeat in controller.
- Use **concrete data** in tests, never generic data.
- If the flow is too long, **split into suites by layer**.
- If critical information is missing (e.g., repository signature), **ask before generating**.
- **Always** run the generated tests and produce coverage output for the tested classes.
- If coverage cannot be calculated due to missing tooling, **explicitly request installation of the required tool**.

## Language and style

- Comments and test names in **Portuguese**.
- Test source code follows the conventions of the project's framework.
