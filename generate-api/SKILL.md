---
name: generate-api
description: Design, generate, implement, refactor, and review production-ready APIs using consistent REST principles, HTTP semantics, resource-oriented architecture, validation, security, error handling, pagination, versioning, documentation, and testing. Use whenever creating or modifying API endpoints, controllers, routes, handlers, services, or API contracts.
---

# generate-api

## Purpose

Build clean, predictable, secure, maintainable, and production-ready APIs.

Follow established REST architectural principles and HTTP semantics rather than creating arbitrary endpoint conventions.

This skill is framework-agnostic and language-agnostic. Adapt its recommendations to the existing project's architecture, technology stack, conventions, and requirements.

Do not introduce unnecessary abstractions, dependencies, or architectural complexity.

## 1. Initial Project Inspection

Before implementing an API:

1. Inspect the existing project structure and architecture.
2. Identify the programming language, framework, database, ORM, and validation library.
3. Identify existing routing, controller, service, repository, and middleware patterns.
4. Inspect existing authentication and authorization mechanisms.
5. Identify existing API response and error-handling conventions.
6. Check whether OpenAPI or another API documentation system is already configured.
7. Reuse existing utilities, abstractions, and conventions whenever appropriate.

Do not introduce a new architecture or replace existing libraries without a good reason.

If the project has no established API conventions, follow the standards defined in this skill.

## 2. Resource-Oriented API Design

Design APIs around resources, not actions.

Use nouns to identify resources and HTTP methods to describe operations.

### Preferred

- GET /users
- GET /users/{id}
- POST /users
- PUT /users/{id}
- PATCH /users/{id}
- DELETE /users/{id}

### Avoid

- GET /getUsers
- POST /createUser
- POST /deleteUser
- POST /updateUser

### Naming conventions

1. Prefer plural nouns for collection resources.
2. Use lowercase resource names.
3. Use kebab-case for multi-word resource names.
4. Use consistent naming conventions across all endpoints.
5. Use resource identifiers to access individual resources.
6. Use nested resources only when there is a meaningful parent-child relationship.

Example:

- /users
- /users/{userId}
- /users/{userId}/orders
- /orders/{orderId}
- /products/{productId}/reviews

Avoid deeply nested resource paths. Prefer a maximum of two or three meaningful resource levels.

For operations that do not naturally map to CRUD, model the operation as a resource or use a clearly documented action endpoint when appropriate.

## 3. HTTP Methods and Semantics

Use HTTP methods according to their intended semantics.

| Method  | Purpose                                     | Safe | Idempotent                |
| ------- | ------------------------------------------- | ---- | ------------------------- |
| GET     | Retrieve resources                          | Yes  | Yes                       |
| HEAD    | Retrieve response headers                   | Yes  | Yes                       |
| OPTIONS | Discover supported communication options    | Yes  | Yes                       |
| POST    | Create resources or initiate processing     | No   | Usually no                |
| PUT     | Create or replace a resource at a known URI | No   | Yes                       |
| PATCH   | Partially modify a resource                 | No   | Depends on implementation |
| DELETE  | Delete a resource                           | No   | Yes                       |
| TRACE   | Diagnostic request                          | Yes  | Yes                       |

### Safety

A safe method must not intentionally modify the server's resource state.

For example:

GET /users/123

GET must not perform hidden state-changing operations such as deleting records, updating balances, or changing account settings.

Logging, analytics, and other incidental server-side effects do not make GET unsafe.

### Idempotency

An operation is idempotent when repeating the same request has the same intended effect on server state as performing it once.

Examples:

- GET: Repeated reads do not modify resource state.
- PUT: Replacing a resource with the same representation repeatedly produces the same intended state.
- DELETE: Repeated deletion does not produce additional deletion effects.
- PATCH: Depends on the operation.
- POST: Usually creates additional resources or effects when repeated.

Do not assume every PATCH operation is non-idempotent or every POST operation must be non-idempotent. Evaluate the actual operation.

### Idempotency keys

For operations where duplicate execution is dangerous, especially payments, checkout, orders, and financial transactions, support idempotency keys where appropriate.

Example:

POST /orders
Idempotency-Key: 8f7a9b2c-unique-key

Implementation requirements:

1. Generate or accept a unique key for each logical operation.
2. Associate the key with the authenticated user or appropriate tenant.
3. Store the operation's status and result.
4. Ensure concurrent requests using the same key cannot execute the operation twice.
5. Return the previously recorded result for legitimate retries.
6. Reject reuse of the same key with a different request payload.
7. Define a retention period for idempotency records.
8. Handle incomplete and failed operations safely.

Do not confuse idempotency with simply checking whether a resource already exists.

## 4. HTTP Status Codes

Use meaningful HTTP status codes to communicate operation outcomes.

### Success

- 200 OK: Successful request with a response body.
- 201 Created: A resource was successfully created.
- 202 Accepted: Request accepted for asynchronous processing.
- 204 No Content: Successful operation without a response body.

### Client errors

- 400 Bad Request: Invalid request structure or malformed input.
- 401 Unauthorized: Missing or invalid authentication.
- 403 Forbidden: Authenticated client lacks permission.
- 404 Not Found: Requested resource does not exist or is intentionally concealed.
- 405 Method Not Allowed: HTTP method is unsupported for the resource.
- 409 Conflict: Conflict with the current resource state.
- 410 Gone: Resource is permanently unavailable.
- 413 Content Too Large: Request body exceeds the supported limit.
- 415 Unsupported Media Type: Unsupported request content type.
- 422 Unprocessable Content: Request is syntactically valid but fails semantic or business validation.
- 429 Too Many Requests: Rate limit exceeded.

### Server errors

- 500 Internal Server Error: Unexpected server-side failure.
- 502 Bad Gateway: Invalid upstream response.
- 503 Service Unavailable: Service temporarily unavailable.
- 504 Gateway Timeout: Upstream service timeout.

Never return 200 OK for an operation that failed.

Choose the status code based on the actual HTTP semantics and application behavior.

Do not expose internal exceptions, stack traces, SQL errors, or infrastructure details to API consumers.

## 5. Consistent Error Responses

Use a predictable error structure across all endpoints.

Prefer the Problem Details format defined by RFC 9457 where appropriate.

Example:

```json
{
  "type": "https://api.example.com/problems/validation-error",
  "title": "Validation Error",
  "status": 422,
  "detail": "The provided request contains invalid fields.",
  "instance": "/users",
  "errors": [
    {
      "field": "email",
      "code": "invalid_format",
      "message": "Please provide a valid email address."
    }
  ]
}
```

Requirements:

1. Keep error response structures consistent.
2. Provide machine-readable error codes.
3. Provide understandable error messages.
4. Include field-level validation errors when relevant.
5. Never expose sensitive internal implementation details.
6. Log detailed exceptions on the server.
7. Return appropriate HTTP status codes.
8. Keep error messages useful without exposing sensitive information.

Avoid creating a different error structure for every endpoint.

## 6. Filtering, Sorting, Searching, and Pagination

Use query parameters to refine collection resources.

Example:

GET /products?category=electronics&status=active

GET /products?search=laptop

GET /products?sortBy=price&sortOrder=asc

GET /products?page=2&limit=20

### Guidelines

- Use query parameters for optional filters.
- Use consistent naming conventions.
- Validate filter and sorting parameters.
- Allowlist sortable fields instead of accepting arbitrary database columns.
- Apply sensible pagination defaults.
- Enforce maximum page sizes.
- Return consistent pagination metadata.
- Avoid exposing database query syntax directly through API parameters.

Example paginated response:

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "limit": 20,
    "totalItems": 100,
    "totalPages": 5,
    "hasNextPage": true,
    "hasPreviousPage": false
  }
}
```

Use cursor-based pagination when it is more appropriate for large datasets, frequently changing collections, or infinite scrolling.

Do not use query parameters to disguise state-changing operations.

## 7. Request Validation and Data Integrity

Validate all incoming data at the API boundary.

Requirements:

1. Validate request bodies, path parameters, and query parameters.
2. Validate data types, formats, lengths, ranges, and required fields.
3. Reject unexpected or unsupported fields where appropriate.
4. Never trust client-provided identifiers, roles, permissions, ownership, or tenant IDs.
5. Enforce business rules in the appropriate application layer.
6. Use database constraints to protect data integrity.
7. Prevent mass-assignment vulnerabilities.
8. Avoid accepting arbitrary objects when a defined schema is expected.

Use the project's existing validation library.

Separate request DTOs or schemas from database models when appropriate.

## 8. Authentication and Authorization

Security must be considered during API design, not added as an afterthought.

### Authentication

- Use established authentication mechanisms.
- Validate tokens and sessions securely.
- Never expose passwords, secrets, or authentication tokens in responses.
- Avoid storing sensitive credentials in logs.
- Apply appropriate expiration and refresh policies.

### Authorization

- Enforce authorization on the server.
- Verify resource ownership before allowing access.
- Apply role-based or attribute-based permissions where appropriate.
- Prevent insecure direct object references (IDOR).
- Never trust frontend-only permission checks.
- Enforce tenant isolation in multi-tenant applications.

### Additional security requirements

- Validate and sanitize untrusted input.
- Apply rate limiting where appropriate.
- Enforce request body size limits.
- Use HTTPS in production.
- Configure CORS intentionally.
- Protect against injection attacks.
- Avoid exposing sensitive information in error responses.
- Use secure pagination and filtering.
- Apply appropriate security headers.

## 9. API Versioning and Backward Compatibility

Design APIs with future evolution in mind.

1. Avoid breaking existing consumers unnecessarily.
2. Adding optional response fields is generally backward-compatible.
3. Removing fields, renaming fields, or changing existing semantics can be breaking.
4. Introduce versioning when a genuine breaking change requires it.
5. Use one consistent versioning strategy.
6. Document deprecated endpoints and fields.
7. Provide migration guidance where appropriate.

Example:

/api/v1/users
/api/v2/users

Do not introduce a new API version for every minor improvement.

Prefer backward-compatible evolution whenever possible.

## 10. Consistent Request and Response Conventions

Maintain consistency across the entire API.

Standardize:

- JSON serialization.
- Property naming conventions.
- Date and timestamp formats.
- Nullability behavior.
- Error response formats.
- Pagination metadata.
- Filtering and sorting parameters.
- Authentication requirements.
- HTTP status codes.
- Content types.
- Resource identifiers.

Use ISO 8601 timestamps and UTC for API timestamps unless the domain requires another explicitly documented representation.

Do not mix naming conventions such as camelCase, snake_case, and kebab-case arbitrarily.

Avoid unnecessary response wrappers or inconsistent nesting.

## 11. API Architecture and Implementation

Follow the architecture already established by the project.

Where appropriate, maintain separation between:

- Routes: Define endpoint paths and HTTP methods.
- Controllers or handlers: Handle HTTP requests and responses.
- Services: Implement business logic.
- Repositories: Handle database access.
- Schemas or DTOs: Validate request and response structures.
- Middleware: Handle authentication, authorization, logging, and other cross-cutting concerns.

Do not put complex business logic directly inside route handlers.

Do not create unnecessary abstraction layers for simple operations.

Use database transactions when multiple related writes must succeed or fail together.

Handle concurrency and race conditions where necessary.

Use asynchronous processing and queues for workloads that should not block HTTP requests.

## 12. API Documentation

Document APIs so developers can integrate without guessing.

Where appropriate, use OpenAPI.

Document:

- Endpoint paths.
- HTTP methods.
- Request parameters.
- Request body schemas.
- Response schemas.
- Success status codes.
- Error status codes.
- Authentication requirements.
- Pagination and filtering.
- Example requests and responses.
- Deprecation and versioning information.

Keep documentation synchronized with actual implementation.

Do not document behavior that the API does not implement.

## 13. API Testing

Test API behavior, not just whether endpoints return a successful response.

Cover:

### Functional tests

- Successful requests.
- Missing required fields.
- Invalid data types.
- Invalid identifiers.
- Missing resources.
- Business rule violations.

### HTTP semantics

- Correct HTTP methods.
- Correct status codes.
- GET does not intentionally mutate resource state.
- PUT and DELETE behave idempotently.
- PATCH behavior matches its documented semantics.
- Repeated POST requests behave according to their contract.
- Idempotency key retries do not duplicate operations.

### Security tests

- Unauthenticated access.
- Unauthorized access.
- Cross-user resource access.
- Cross-tenant resource access.
- Invalid or expired tokens.
- Input injection attempts.

### Reliability tests

- Database failures.
- External service failures.
- Concurrent requests.
- Timeout and retry scenarios.
- Transaction rollback behavior.

Use the testing framework already established by the project.

## 14. API Generation Workflow

Whenever generating or modifying an API, follow this workflow:

### Step 1: Understand requirements

Identify:

- Resource or domain entity.
- Required operations.
- Relationships.
- Authentication requirements.
- Authorization rules.
- Business constraints.
- Expected request and response formats.

### Step 2: Design the API contract

Define:

- Resource-oriented endpoint paths.
- HTTP methods.
- Path and query parameters.
- Request schemas.
- Response schemas.
- Status codes.
- Error responses.
- Pagination and filtering behavior.

### Step 3: Review HTTP semantics

Check:

- Is the endpoint using the correct HTTP method?
- Is the operation safe or unsafe?
- Is it idempotent?
- Can clients safely retry it?
- Does it require an idempotency key?
- Are status codes appropriate?

### Step 4: Implement

Follow existing architecture and project conventions.

Implement validation, authorization, business logic, persistence, and error handling.

### Step 5: Test

Add relevant functional, security, integration, and HTTP semantics tests.

### Step 6: Document

Update API documentation and provide request and response examples.

### Step 7: Review

Before considering the API complete, verify:

- Resource-oriented naming.
- Consistent endpoint conventions.
- Correct HTTP method semantics.
- Safety and idempotency.
- Appropriate HTTP status codes.
- Consistent error structures.
- Input validation.
- Authentication and authorization.
- Pagination and filtering.
- Backward compatibility.
- Test coverage.
- Documentation accuracy.

## 15. Important Rules

- Never invent custom HTTP semantics when standard semantics are appropriate.
- Never use GET for state-changing operations.
- Never return success status codes for failed operations.
- Never expose internal exceptions to API consumers.
- Never trust client-provided authorization or ownership data.
- Never assume a retry is harmless.
- Never assume PATCH is automatically idempotent.
- Never assume POST is automatically protected against duplicate execution.
- Never sacrifice backward compatibility without considering existing consumers.
- Never introduce unnecessary complexity just to follow a pattern.
- Always prioritize correctness, security, consistency, and maintainability.

## Expected Outcome

Every API generated using this skill should be:

1. Predictable and intuitive.
2. Consistent across endpoints.
3. Semantically correct according to HTTP standards.
4. Secure by default.
5. Resilient to network failures and retries.
6. Easy to integrate with frontend and external clients.
7. Easy to test and document.
8. Maintainable and extensible.

The ultimate goal is not merely to generate working endpoints.

## Naming Conventions

Follow these conventions consistently across the API:

- URL paths: kebab-case, using plural resource names.
- Query parameters: camelCase.
- Request body properties: camelCase.
- Response body properties: camelCase.
- JSON nested properties: camelCase.
- Database tables: snake_case.
- Database columns: snake_case.
- TypeScript variables and functions: camelCase.
- TypeScript classes and types: PascalCase.
- Environment variables: SCREAMING_SNAKE_CASE.

Examples:

GET /order-items?pageSize=20&sortBy=createdAt

POST /order-items

```json
{
  "productId": "prod_123",
  "quantity": 2
}
```

Response:

```json
{
  "id": "item_123",
  "productId": "prod_123",
  "quantity": 2,
  "createdAt": "2026-11-05T12:00:00Z"
}
```

Use kebab-case for URLs, camelCase for everything in JSON and query parameters, and snake_case for database identifiers.

- Keep API naming conventions independent of database naming conventions.
- Never mix naming conventions within the same API contract.

**The goal is to generate APIs whose design makes the correct way to use them obvious.**
