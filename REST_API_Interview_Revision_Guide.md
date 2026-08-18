# REST API & General API Concepts — Fresher Software Engineering Interview Revision Guide

## 1. Fundamentals

### Q1. What is an API?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Whether you understand the fundamental purpose of an API.

**Expected Answer:**

An API, or Application Programming Interface, is a defined interface through which one software component can interact with another.

For web APIs, a client sends a request to a server and the server returns a response.

```text
Frontend
   |
   | GET /users/123
   v
Backend API
   |
   | JSON response
   v
Frontend
```

The API defines available operations, inputs, outputs, authentication requirements, and error behavior.

**Key Points to Remember:**
- API = interface/contract between software components.
- APIs are not limited to HTTP.
- A REST API is one style of designing web APIs.

---

### Q2. What is a REST API?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Whether you understand REST rather than equating REST with "JSON over HTTP."

**Expected Answer:**

REST stands for **Representational State Transfer**. It is an architectural style for designing networked applications.

A RESTful API models resources and uses standard HTTP semantics to operate on those resources.

```text
GET    /users/42       → retrieve user
POST   /users          → create user
PUT    /users/42       → replace user
PATCH  /users/42       → partially update user
DELETE /users/42       → delete user
```

REST commonly uses HTTP and JSON, but REST itself is not synonymous with either one.

**Key Points to Remember:**
- REST is an architectural style.
- Resources are central.
- HTTP methods and status codes provide standard semantics.
- JSON is common but not a REST requirement.

---

### Q3. What are the main principles/constraints of REST?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you understand REST conceptually.

**Expected Answer:**

The commonly discussed REST constraints are:

1. Client-server separation.
2. Stateless communication.
3. Cacheability.
4. Uniform interface.
5. Layered system.
6. Code-on-demand, which is optional.

Statelessness means the server does not depend on stored client session state to understand a request; the request should contain the information needed to process it.

**Key Points to Remember:**
- Statelessness is especially important in interviews.
- Code-on-demand is optional.
- REST is broader than CRUD.

---

### Q4. What is a resource in REST?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Understanding of REST's resource-oriented model.

**Expected Answer:**

A resource is a conceptual entity or collection exposed by an API.

Examples:

```text
/users
/users/42
/products
/orders/1001
```

The URL identifies the resource, while the HTTP method describes the requested operation.

**Key Points to Remember:**
- URL identifies a resource.
- HTTP method expresses the operation.
- Prefer meaningful resource-oriented URLs.

---

### Q5. What is the difference between an API, REST API, and web service?

**Priority:** 🟠 Important  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you understand the relationship between these terms.

**Expected Answer:**

An **API** is a general interface through which software components communicate.

A **web API** is an API exposed over a network, commonly using HTTP.

A **REST API** is a web API designed according to REST architectural principles.

A **web service** generally refers to a network-accessible software service and can use different technologies, including REST or SOAP.

**Key Points to Remember:**
- API is the broadest concept.
- REST is an architectural style.
- HTTP is commonly used for web APIs.

---

## 2. HTTP Methods and Semantics

### Q6. Explain GET, POST, PUT, PATCH, and DELETE.

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Core REST/HTTP knowledge.

**Expected Answer:**

| Method | Typical purpose |
|---|---|
| GET | Retrieve a resource |
| POST | Create a resource or trigger a non-idempotent operation |
| PUT | Replace a resource at a known URI |
| PATCH | Partially modify a resource |
| DELETE | Remove a resource |

Example:

```text
GET    /users/10
POST   /users
PUT    /users/10
PATCH  /users/10
DELETE /users/10
```

**Key Points to Remember:**
- PUT generally represents replacement.
- PATCH is intended for partial modification.
- POST is commonly used for creation when the server chooses the resource URI.

---

### Q7. What is the difference between PUT and PATCH?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** A common REST interview distinction.

**Expected Answer:**

`PUT` is generally used to replace the representation of a resource at a target URI.

`PATCH` is used to apply partial modifications.

```http
PUT /users/10
{
  "name": "Aditya",
  "email": "a@example.com",
  "age": 22
}
```

Whereas:

```http
PATCH /users/10
{
  "age": 23
}
```

The exact API contract can define required fields, but the semantic distinction is replacement versus partial modification.

**Key Points to Remember:**
- PUT → replacement.
- PATCH → partial modification.
- They are not interchangeable.

---

### Q8. What does idempotent mean in HTTP?

**Priority:** 🔴 Must Know  
**Difficulty:** Tricky

**What the interviewer is testing:** Understanding of HTTP semantics and retries.

**Expected Answer:**

An operation is idempotent if making the same request multiple times has the same intended effect on server state as making it once.

For example, setting a resource to a particular representation with `PUT` can be idempotent.

`POST` is generally not idempotent because repeating it may create multiple resources.

Idempotency does not mean every response will be byte-for-byte identical.

**Key Points to Remember:**
- GET, PUT, and DELETE are defined as idempotent.
- POST is generally non-idempotent.
- It is about intended effect, not identical responses.

---

### Q9. What is the difference between safe and idempotent HTTP methods?

**Priority:** 🟠 Important  
**Difficulty:** Tricky

**What the interviewer is testing:** Whether you know two related but different HTTP concepts.

**Expected Answer:**

A **safe** method is intended primarily for retrieving information and should not request a state-changing action.

`GET`, `HEAD`, and `OPTIONS` are safe.

An **idempotent** method can be repeated without changing the intended result beyond what the first request did.

```text
GET     → safe + idempotent
PUT     → idempotent, not safe
DELETE  → idempotent, not safe
POST    → generally neither
```

**Key Points to Remember:**
- Safe ≠ idempotent.
- A safe request should not intentionally modify server state.
- An idempotent request can still modify state.

---

## 3. HTTP Request and Response

### Q10. What does an HTTP request contain?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Understanding of API communication.

**Expected Answer:**

An HTTP request commonly contains:

```text
Request line
Headers
Blank line
Optional body
```

Example:

```http
POST /users HTTP/1.1
Content-Type: application/json
Authorization: Bearer <token>

{
  "name": "John"
}
```

The request line contains the method, target, and HTTP version. Headers provide metadata, and the body carries optional content.

**Key Points to Remember:**
- Method.
- URL/path.
- Headers.
- Optional body.
- Query parameters are part of the target URL.

---

### Q11. What does an HTTP response contain?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Basic HTTP response knowledge.

**Expected Answer:**

An HTTP response generally contains:

```text
Status line
Headers
Blank line
Optional body
```

Example:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 10,
  "name": "John"
}
```

The status code communicates the broad result, headers provide metadata, and the body can contain the representation or error information.

**Key Points to Remember:**
- Status code.
- Response headers.
- Optional response body.
- Content-Type tells the client how to interpret the body.

---

### Q12. What are HTTP status code categories?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Ability to communicate API results correctly.

**Expected Answer:**

| Range | Meaning |
|---|---|
| 1xx | Informational |
| 2xx | Successful |
| 3xx | Redirection |
| 4xx | Client/request errors |
| 5xx | Server errors |

Common API codes:

```text
200 OK
201 Created
204 No Content

400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Content

500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
```

**Key Points to Remember:**
- 401 → authentication is required/failed.
- 403 → access is not allowed.
- 404 → requested resource/route was not found.
- 5xx generally indicates server-side failure.

---

### Q13. What is the difference between 401 and 403?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Authentication vs authorization.

**Expected Answer:**

`401 Unauthorized` indicates that the request lacks valid authentication credentials or authentication has not succeeded.

`403 Forbidden` means the server understood the request but will not authorize the operation.

```text
401 → Authentication problem
403 → Authorization/access problem
```

**Key Points to Remember:**
- 401 → authentication.
- 403 → authorization.
- Don't use them interchangeably.

---

### Q14. When would you use 200, 201, and 204?

**Priority:** 🟠 Important  
**Difficulty:** Basic

**What the interviewer is testing:** Correct status-code semantics.

**Expected Answer:**

- `200 OK` → successful request with a response/result.
- `201 Created` → a new resource was successfully created.
- `204 No Content` → request succeeded but there is no response body.

For example:

```text
POST /users       → 201 Created
GET /users/10     → 200 OK
DELETE /users/10  → often 204 No Content
```

**Key Points to Remember:**
- 201 is appropriate for successful creation.
- 204 means no response content.
- Status codes are part of the API contract.

---

## 4. API Design

### Q15. What makes an API well designed?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** API design judgment.

**Expected Answer:**

A well-designed API should have:

- Clear and consistent resource naming.
- Correct HTTP method semantics.
- Meaningful status codes.
- Consistent request/response schemas.
- Validation and predictable errors.
- Authentication and authorization where required.
- Pagination for large collections.
- Versioning when breaking changes are necessary.
- Documentation.
- Appropriate caching and rate limiting.

The client should be able to understand and use the API without guessing its behavior.

**Key Points to Remember:**
- Consistency is critical.
- An API is a contract.
- Predictability matters.

---

### Q16. Should REST URLs use nouns or verbs?

**Priority:** 🟠 Important  
**Difficulty:** Basic

**What the interviewer is testing:** REST resource modeling.

**Expected Answer:**

REST-style URLs generally use nouns representing resources, while the HTTP method expresses the action.

Prefer:

```text
GET /users/42
POST /users
DELETE /users/42
```

rather than:

```text
GET /getUser/42
POST /createUser
POST /deleteUser
```

There are cases where an operation is not naturally CRUD-like, but resource-oriented naming should be the default.

**Key Points to Remember:**
- Nouns in resource paths.
- HTTP method expresses common operations.
- Avoid redundant verbs in ordinary resource endpoints.

---

### Q17. What is API versioning, and why is it needed?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** API evolution and backward compatibility.

**Expected Answer:**

API versioning allows a server to introduce breaking changes without unexpectedly breaking existing clients.

A common approach is URL versioning:

```text
/api/v1/users
/api/v2/users
```

Other approaches include version information in headers or content negotiation.

Versioning is mainly needed when the API contract changes in a backward-incompatible way.

**Key Points to Remember:**
- Avoid breaking existing clients unnecessarily.
- Versioning is about API contract compatibility.
- Use a consistent strategy.

---

### Q18. What is pagination, and why is it important?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** API scalability and response design.

**Expected Answer:**

Pagination limits how many records an API returns in one response.

```text
GET /products?page=2&limit=20
```

Instead of returning a million products, the server returns a manageable subset.

Common approaches include offset/page pagination and cursor-based pagination.

**Key Points to Remember:**
- Prevents huge responses.
- Reduces memory, network, and database work.
- Cursor pagination can be useful for large/dynamic datasets.

---

### Q19. What is the difference between offset pagination and cursor pagination?

**Priority:** 🟠 Important  
**Difficulty:** Tricky

**What the interviewer is testing:** Pagination trade-offs.

**Expected Answer:**

**Offset pagination:**

```text
GET /items?offset=100&limit=20
```

It is simple but large offsets can become inefficient, and concurrent inserts/deletes can cause records to shift between pages.

**Cursor pagination:**

```text
GET /items?limit=20&after=<cursor>
```

The cursor represents a position in an ordered result set. It can provide more stable and efficient pagination for large, changing datasets when backed by an appropriate index/order.

**Key Points to Remember:**
- Offset → simple.
- Cursor → often better for large/high-churn collections.
- Cursor pagination requires stable ordering.

---

## 5. Authentication, Security, and Errors

### Q20. What is the difference between authentication and authorization?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Fundamental API security knowledge.

**Expected Answer:**

**Authentication** verifies who the client/user is.

**Authorization** determines what that authenticated identity is allowed to do.

```text
Login/token verification
       ↓
Authentication
       ↓
Identify user
       ↓
Authorization
       ↓
Check permission
```

A user can be authenticated but still lack permission for a particular operation.

**Key Points to Remember:**
- Authentication → identity.
- Authorization → permissions.
- Never rely only on frontend authorization checks.

---

### Q21. What are common API authentication mechanisms?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Familiarity with authentication approaches.

**Expected Answer:**

Common mechanisms include:

- API keys.
- Session-based authentication.
- OAuth 2.0.
- Bearer tokens.
- JWT-based access tokens.
- Mutual TLS in some service-to-service environments.

The appropriate mechanism depends on the application.

HTTPS is essential for protecting credentials/tokens in transit.

**Key Points to Remember:**
- Mechanism depends on use case.
- JWT is a token format, not a complete authentication architecture.
- Credentials should be protected in transit and at rest where applicable.

---

### Q22. How should an API handle errors?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** API reliability and consistency.

**Expected Answer:**

An API should return an appropriate HTTP status code and a consistent error structure.

```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User was not found",
    "requestId": "abc123"
  }
}
```

The exact schema should be consistent across endpoints.

Internal stack traces, database errors, secrets, and implementation details should not be exposed to clients.

**Key Points to Remember:**
- Use meaningful status codes.
- Use consistent error formats.
- Give clients actionable information.
- Don't expose sensitive internal details.

---

## 6. Performance and Advanced Concepts

### Q23. What is caching in an API, and how does HTTP caching work?

**Priority:** 🟠 Important  
**Difficulty:** Intermediate

**What the interviewer is testing:** Performance and HTTP semantics.

**Expected Answer:**

Caching stores a response so future requests can be served without repeating the full backend computation.

HTTP provides headers such as:

```http
Cache-Control: max-age=3600
ETag: "abc123"
Last-Modified: ...
```

An `ETag` can support conditional requests:

```text
Client → If-None-Match: "abc123"
Server → 304 Not Modified
```

This avoids retransmitting unchanged content.

**Key Points to Remember:**
- Cacheability should be explicit.
- HTTP caching can reduce latency and server load.
- `ETag` supports conditional requests.
- Don't cache sensitive/user-specific responses incorrectly.

---

### Q24. What is rate limiting, and why is it used?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** API protection and resource management.

**Expected Answer:**

Rate limiting restricts how many requests a client can make according to a defined policy.

For example:

```text
100 requests / minute / user
```

It helps prevent abuse, protect backend resources, reduce accidental overload, and control expensive operations.

Common algorithms include token bucket and leaky bucket.

An API can return `429 Too Many Requests` when a limit is exceeded.

**Key Points to Remember:**
- Protects availability.
- 429 is commonly used when limits are exceeded.
- Limits can be per user, IP, API key, tenant, or endpoint.

---

### Q25. What are common REST API mistakes and best practices?

**Priority:** 🔴 Must Know  
**Difficulty:** Tricky

**What the interviewer is testing:** Whether you can defend an API implementation on your resume.

**Expected Answer:**

Common mistakes include:

1. Using verbs in every URL instead of modeling resources.
2. Returning `200 OK` for every situation.
3. Confusing `401` and `403`.
4. Returning huge unpaginated collections.
5. Ignoring idempotency when implementing retryable operations.
6. Exposing database implementation details directly in API contracts.
7. Inconsistent response/error formats.
8. Missing input validation.
9. Putting authorization logic only in the frontend.
10. Exposing secrets or sensitive data.
11. No rate limiting on sensitive/expensive endpoints.
12. Breaking existing clients without a compatibility/versioning strategy.
13. Ignoring caching opportunities.
14. Not documenting API behavior.

A strong API should be consistent, predictable, secure, documented, and designed around clear contracts.

**Key Points to Remember:**
- HTTP semantics matter.
- Treat an API as a public contract.
- Design for failures and retries.
- Validate input and enforce authorization server-side.
- Consider pagination, caching, rate limiting, and versioning.

---

# Final Revision Section

## 1. 🔴 Must-Know Questions

1. What is an API?
2. What is REST?
3. What are REST's main constraints?
4. What is a REST resource?
5. Explain GET, POST, PUT, PATCH, DELETE.
6. PUT vs PATCH?
7. What is idempotency?
8. Safe vs idempotent methods?
9. Structure of an HTTP request?
10. Structure of an HTTP response?
11. HTTP status-code categories?
12. 401 vs 403?
13. 200 vs 201 vs 204?
14. What makes an API well designed?
15. What is API versioning?
16. What is pagination?
17. Authentication vs authorization?
18. Common API authentication mechanisms?
19. How should API errors be designed?
20. What is caching?
21. What is rate limiting?
22. What are common REST API mistakes?

---

## 2. ⚠️ Tricky Concepts

### REST ≠ JSON

REST is an architectural style. JSON is simply a commonly used representation format.

### REST ≠ CRUD

CRUD is a data-operation model. REST uses HTTP semantics and resource representations more broadly.

### PUT ≠ PATCH

PUT generally replaces a resource representation, while PATCH applies partial modifications.

### Idempotent ≠ safe

```text
GET     → safe + idempotent
PUT     → idempotent, not safe
DELETE  → idempotent, not safe
POST    → generally neither
```

### 401 ≠ 403

```text
401 → authentication failed/required
403 → authenticated/recognized but not allowed
```

### Stateless ≠ "the server stores no data"

REST statelessness means the server does not depend on stored client session state to understand each request. The server can still store application data in databases.

### JWT ≠ authentication system

JWT is a token format. Authentication includes issuing, validating, expiring, and managing credentials/tokens.

### Pagination is not only a frontend feature

It is a backend/API/database design decision affecting response size, query cost, network usage, and user experience.

---

# 3. 🧠 Quick Revision Sheet

## REST Request Flow

```text
Client
  ↓
HTTP Request
  ├── Method
  ├── URL
  ├── Headers
  └── Body
  ↓
API Server
  ↓
Authentication
  ↓
Authorization
  ↓
Validation
  ↓
Business Logic
  ↓
Database / Services
  ↓
HTTP Response
  ├── Status Code
  ├── Headers
  └── Body
  ↓
Client
```

## HTTP Methods

| Method | Typical use | Idempotent | Safe |
|---|---|---:|---:|
| GET | Retrieve | Yes | Yes |
| POST | Create/action | Generally No | No |
| PUT | Replace | Yes | No |
| PATCH | Partial update | Not inherently guaranteed | No |
| DELETE | Delete | Yes | No |
| HEAD | Headers only | Yes | Yes |
| OPTIONS | Capabilities/preflight | Yes | Yes |

> PATCH is not inherently required to be idempotent; a particular PATCH operation can be designed to be idempotent.

## Important Status Codes

```text
200 → OK
201 → Created
204 → No Content

400 → Bad Request
401 → Authentication problem
403 → Not allowed
404 → Not Found
409 → Conflict
422 → Semantically invalid request content

429 → Too Many Requests

500 → Internal Server Error
502 → Bad Gateway
503 → Service Unavailable
```

## API Design Checklist

```text
Resource-oriented URLs
        ↓
Correct HTTP methods
        ↓
Correct status codes
        ↓
Validation
        ↓
Authentication
        ↓
Authorization
        ↓
Consistent errors
        ↓
Pagination
        ↓
Caching
        ↓
Rate limiting
        ↓
Documentation
        ↓
Versioning when needed
```

---

# 4. 🔑 Important Terms

Be ready to recognize and explain:

- API
- Web API
- REST
- RESTful
- Resource
- Representation
- Endpoint
- URI
- URL
- HTTP
- HTTP method
- GET
- POST
- PUT
- PATCH
- DELETE
- Idempotency
- Safe method
- Statelessness
- Client-server architecture
- Cacheability
- Uniform interface
- Layered system
- HTTP headers
- Request body
- Response body
- Content-Type
- Accept
- Authorization
- Authentication
- Bearer token
- API key
- OAuth 2.0
- JWT
- Status code
- 2xx
- 4xx
- 5xx
- Pagination
- Cursor
- Offset
- Rate limiting
- Token bucket
- Caching
- ETag
- Conditional request
- API versioning
- Backward compatibility
- Validation
- Serialization
- Deserialization
- CORS
- API Gateway
- Retry
- Timeout
- Circuit breaker

---

# 5. 🎯 Interview Tips

### 1. Don't define REST as "an API that uses JSON"

A better answer is:

> "REST is an architectural style for networked applications that emphasizes resources, a uniform interface, stateless communication, cacheability, and standard HTTP semantics when implemented over HTTP."

### 2. Always know HTTP method semantics

If you mention REST on your resume, expect:

> "Why would you use PATCH instead of PUT?"

and:

> "Is POST idempotent?"

### 3. Know status codes beyond 200 and 404

At minimum:

```text
200
201
204
400
401
403
404
409
422
429
500
502
503
```

### 4. Explain API design as a contract

A strong statement is:

> "I treat an API as a contract between client and server. That means I care about consistent request/response schemas, status codes, validation, authentication, authorization, error formats, backward compatibility, and documentation."

### 5. Be ready to defend your own APIs

If REST API is on your resume, expect:

- What endpoints did you create?
- Why did you choose those HTTP methods?
- What were your request and response schemas?
- How did you handle authentication?
- How did you handle authorization?
- How did you validate input?
- What status codes did you return?
- How did you handle errors?
- Did you implement pagination?
- How did you prevent abuse?
- How did you document the API?
- How would you version it?
- How would you improve its performance?

### 6. Think about failure, not only the happy path

```text
Invalid input
     ↓
Unauthorized user
     ↓
Resource not found
     ↓
Conflict
     ↓
Rate limit exceeded
     ↓
Database failure
     ↓
Downstream service failure
     ↓
Timeout
```

An interviewer often uses these cases to distinguish a basic API implementation from production-oriented thinking.

---

# 6. ❓ Self-Test

Try answering these aloud without looking at the answers:

1. Explain an API in 30 seconds.
2. What is REST?
3. REST vs RESTful?
4. What is a resource?
5. What makes an API RESTful?
6. Explain GET, POST, PUT, PATCH, and DELETE.
7. PUT vs PATCH?
8. What is idempotency?
9. Which HTTP methods are idempotent?
10. Safe vs idempotent?
11. What is statelessness?
12. Does stateless mean the server cannot use a database?
13. What does an HTTP request contain?
14. What does an HTTP response contain?
15. Explain 200, 201, and 204.
16. Explain 400, 401, 403, 404, 409, and 422.
17. Why is 401 different from 403?
18. What is API versioning?
19. Offset vs cursor pagination?
20. What is authentication?
21. What is authorization?
22. JWT vs session-based authentication?
23. What is API caching?
24. What is rate limiting?
25. If you had to review a REST API before production deployment, what would you check?

---

# Final Interview Target

For a fresher Software Engineering interview, use this structure:

> **Definition → HTTP/technical mechanism → example → trade-off or pitfall**

For example:

> "A REST API is an API designed around REST principles, where resources are identified by URIs and standard HTTP methods express operations on those resources. For example, `GET /users/42` retrieves a user and `PATCH /users/42` partially updates one. A key REST property is statelessness, meaning each request contains the information needed to process it rather than relying on server-side client session state. In a production API, I would also consider authentication, authorization, validation, consistent errors, pagination, rate limiting, caching, and backward compatibility."

That style demonstrates **actual API understanding rather than memorized CRUD terminology**.
