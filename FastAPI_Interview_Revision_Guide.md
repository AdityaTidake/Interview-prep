# FastAPI — Fresher Software Engineering Interview Revision Guide

> **Goal:** Be able to confidently defend FastAPI as a backend framework on your resume, including its architecture, request lifecycle, routing, validation, dependency injection, asynchronous execution, authentication, performance, testing, and common pitfalls.

---

# 1. Fundamentals

### Q1. What is FastAPI?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Whether you understand what FastAPI is and why it is used.

**Expected Answer:**

FastAPI is a modern Python web framework for building APIs. It is built around Python type hints and provides automatic request validation, serialization, interactive API documentation, dependency injection, and support for asynchronous programming.

It commonly uses **Starlette** for the web/HTTP layer and **Pydantic** for data validation and serialization.

A simple API looks like:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/hello")
def hello():
    return {"message": "Hello"}
```

**Key Points to Remember:**
- Python framework for APIs.
- Type-hint driven.
- Uses Starlette for web capabilities.
- Uses Pydantic for validation/data models.
- Supports both `def` and `async def` endpoints.
- Automatically generates OpenAPI documentation.

---

### Q2. Why would you choose FastAPI over Flask or Django?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you understand FastAPI's strengths rather than choosing it because it is popular.

**Expected Answer:**

I would choose FastAPI when I primarily need a modern API service with strong type-based validation, automatic OpenAPI documentation, and asynchronous support.

Compared with Flask, FastAPI provides more built-in API-oriented features such as request validation and OpenAPI generation.

Django is a broader web framework with features such as ORM, authentication, admin, templates, and many batteries-included capabilities. FastAPI is generally more focused on API services and lets me choose supporting components separately.

**Key Points to Remember:**
- FastAPI → API-focused, typed, validation, OpenAPI, async.
- Flask → lightweight and flexible, fewer built-in API conventions.
- Django → full-featured web framework.
- Choice depends on application requirements.

---

### Q3. What are the main components behind FastAPI?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you know FastAPI's underlying ecosystem.

**Expected Answer:**

The important pieces are:

```text
FastAPI
   │
   ├── Starlette → web/HTTP layer
   │       ├── routing
   │       ├── middleware
   │       ├── requests/responses
   │       └── WebSockets
   │
   └── Pydantic → data validation/serialization
```

FastAPI also integrates with ASGI servers such as Uvicorn for running the application.

**Key Points to Remember:**
- FastAPI is built on Starlette.
- Pydantic handles data validation and serialization.
- Uvicorn is commonly used as the ASGI server.
- FastAPI itself is the framework tying these capabilities together.

---

### Q4. What is ASGI, and why is it important for FastAPI?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you understand the Python web execution model used by FastAPI.

**Expected Answer:**

ASGI stands for **Asynchronous Server Gateway Interface**. It is a Python standard/interface between an asynchronous-capable web server and an application.

FastAPI applications are ASGI applications. An ASGI server such as Uvicorn receives network requests and communicates with the FastAPI application through the ASGI interface.

Conceptually:

```text
Client
  ↓
Uvicorn / ASGI server
  ↓
FastAPI application
  ↓
Route
  ↓
Business logic
  ↓
Response
```

ASGI also supports protocols beyond ordinary HTTP, including WebSockets.

**Key Points to Remember:**
- ASGI is an interface/specification.
- Uvicorn is an ASGI server.
- FastAPI is an ASGI application/framework.
- ASGI enables asynchronous web applications.

---

### Q5. What is the difference between Uvicorn and FastAPI?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Whether you can distinguish the framework from the server.

**Expected Answer:**

FastAPI is the web framework used to define routes, validation, dependencies, and application logic.

Uvicorn is an **ASGI server** that runs the FastAPI application and handles communication with clients.

For example:

```bash
uvicorn main:app --reload
```

Here:
- `main` is the Python module.
- `app` is the FastAPI application object.
- Uvicorn runs that ASGI application.

**Key Points to Remember:**
- FastAPI → application framework.
- Uvicorn → server.
- They work together but are not the same thing.

---

# 2. Routing and Requests

### Q6. How do you define a route in FastAPI?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Basic FastAPI usage.

**Expected Answer:**

Routes are defined using decorators such as `@app.get()`, `@app.post()`, `@app.put()`, and `@app.delete()`.

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}
```

The decorator associates an HTTP method and path with a Python function.

FastAPI uses the function signature and type hints to understand parameters and validate incoming data.

**Key Points to Remember:**
- Decorators define routes.
- HTTP methods represent different operations.
- Type hints help FastAPI validate parameters.

---

### Q7. What is the difference between path parameters and query parameters in FastAPI?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Request parameter handling.

**Expected Answer:**

A **path parameter** is part of the URL path:

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    ...
```

A **query parameter** appears after `?`:

```text
/users?limit=10&active=true
```

and can be defined as:

```python
@app.get("/users")
def get_users(limit: int = 10, active: bool = True):
    ...
```

FastAPI determines where parameters come from based on the route and function signature.

**Key Points to Remember:**
- Path → `/users/{id}`
- Query → `/users?id=10`
- Type hints provide validation/conversion.

---

### Q8. How does FastAPI handle request bodies?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Understanding of API request validation.

**Expected Answer:**

FastAPI commonly uses Pydantic models to define the expected request body.

```python
from pydantic import BaseModel

class UserCreate(BaseModel):
    name: str
    age: int

@app.post("/users")
def create_user(user: UserCreate):
    return user
```

FastAPI reads the incoming JSON body, validates it against the model, and provides the parsed model instance to the endpoint.

If the input does not satisfy the schema, FastAPI returns a validation error response instead of passing invalid data to the endpoint.

**Key Points to Remember:**
- Pydantic models define structured request data.
- Validation happens before the route handler runs.
- Type annotations contribute to schema generation.

---

### Q9. How does FastAPI automatically generate API documentation?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of FastAPI's OpenAPI integration.

**Expected Answer:**

FastAPI uses route definitions, Python type hints, Pydantic models, and metadata to generate an **OpenAPI schema**.

It exposes interactive documentation by default, commonly at:

```text
/docs       → Swagger UI
/redoc      → ReDoc
```

The OpenAPI schema itself is also available through the application.

This means the API contract can be inspected and tested directly through the browser.

**Key Points to Remember:**
- FastAPI generates OpenAPI documentation automatically.
- Swagger UI and ReDoc provide interactive documentation.
- Type hints and Pydantic models contribute to the generated schema.

---

# 3. Pydantic and Validation

### Q10. What is Pydantic, and why is it important in FastAPI?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of FastAPI's validation layer.

**Expected Answer:**

Pydantic is a Python library for defining data models and validating/parsing data using Python type annotations.

FastAPI uses Pydantic heavily for:
- Request body validation.
- Response serialization/schema definition.
- Configuration in related FastAPI ecosystems.
- Generating OpenAPI schemas from models.

Example:

```python
class Product(BaseModel):
    name: str
    price: float
    quantity: int
```

FastAPI can use this model to validate incoming JSON.

**Key Points to Remember:**
- Pydantic validates and parses structured data.
- It works closely with Python type hints.
- FastAPI uses it for API data models.

---

### Q11. What happens when invalid request data is sent to a FastAPI endpoint?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you understand request validation.

**Expected Answer:**

FastAPI validates incoming parameters and request bodies before executing the route handler.

For example, if:

```python
class User(BaseModel):
    age: int
```

expects an integer but the request contains data that cannot be validated according to the model, FastAPI generates a validation error response.

The important point is that invalid input is rejected at the API boundary, so application logic does not have to manually validate every field.

**Key Points to Remember:**
- Validation occurs before normal route execution.
- Validation errors are represented as structured HTTP errors.
- Strong schemas improve reliability and API contracts.

---

### Q12. What is the difference between request models and response models?

**Priority:** 🟠 Important  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you understand API data contracts.

**Expected Answer:**

A request model defines what the client is allowed or expected to send.

A response model defines the structure of data the API returns.

For example:

```python
class UserCreate(BaseModel):
    name: str
    password: str

class UserResponse(BaseModel):
    id: int
    name: str

@app.post("/users", response_model=UserResponse)
def create_user(user: UserCreate):
    ...
```

This prevents sensitive request-only fields such as passwords from accidentally becoming part of the response contract.

**Key Points to Remember:**
- Request model → input contract.
- Response model → output contract.
- Separate models improve security and API clarity.

---

# 4. Dependency Injection

### Q13. What is dependency injection in FastAPI?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** One of FastAPI's most important framework features.

**Expected Answer:**

FastAPI's dependency injection system lets an endpoint declare the services or data it depends on, and FastAPI resolves those dependencies before calling the endpoint.

```python
from fastapi import Depends

def get_database():
    return database

@app.get("/users")
def get_users(db = Depends(get_database)):
    ...
```

Dependencies can be used for:
- Database sessions.
- Authentication.
- Authorization.
- Common query parameters.
- Shared business logic.
- Configuration.

This makes code easier to reuse and test.

**Key Points to Remember:**
- `Depends()` declares dependencies.
- FastAPI resolves them automatically.
- Dependencies can themselves have dependencies.
- Useful for authentication and database access.

---

### Q14. How would you implement authentication using FastAPI dependencies?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Practical understanding of dependency injection.

**Expected Answer:**

I can create a dependency that extracts and validates authentication credentials, then inject the authenticated user into protected endpoints.

Conceptually:

```text
Request
  ↓
Authentication dependency
  ↓
Validate token
  ↓
Find user
  ↓
Inject current user
  ↓
Protected endpoint
```

For example, FastAPI's security utilities can work with OAuth2 bearer tokens. The actual token verification logic can then validate a JWT or another credential according to the application's authentication design.

**Key Points to Remember:**
- Authentication can be centralized in dependencies.
- Authorization can be layered on top.
- Never trust user-provided identity without verifying credentials.
- Keep secrets and signing keys outside source code.

---

# 5. Async Programming

### Q15. What is the difference between `def` and `async def` endpoints in FastAPI?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you understand FastAPI's asynchronous execution model.

**Expected Answer:**

A normal `def` endpoint is synchronous. FastAPI can execute it using a thread pool so that synchronous work does not directly block the event loop.

An `async def` endpoint is a coroutine that can use `await` for asynchronous operations.

```python
@app.get("/users")
async def get_users():
    data = await async_database_call()
    return data
```

The important rule is:

> Use `async def` when the operations inside the endpoint are genuinely awaitable/asynchronous.

Simply changing `def` to `async def` does not make blocking code asynchronous.

**Key Points to Remember:**
- `async def` → coroutine.
- `await` allows cooperative suspension while waiting.
- Blocking synchronous code inside `async def` can still block the event loop.
- FastAPI supports both styles.

---

### Q16. Does using `async def` automatically make a FastAPI application faster?

**Priority:** 🔴 Must Know  
**Difficulty:** Tricky

**What the interviewer is testing:** Whether you understand asynchronous programming rather than treating `async` as a magic performance switch.

**Expected Answer:**

No. `async def` is useful when the workload contains asynchronous operations that spend time waiting, such as network I/O.

For example:

```python
async def endpoint():
    result = await async_db_call()
    return result
```

While the database call is waiting, the event loop can work on other tasks.

But if I do CPU-heavy or blocking synchronous work inside the coroutine, the event loop can still be blocked.

**Key Points to Remember:**
- Async improves concurrency for suitable I/O workloads.
- It does not make CPU calculations faster.
- Blocking libraries can undermine an async design.

---

### Q17. What happens if you call a blocking function inside an `async def` FastAPI endpoint?

**Priority:** 🔴 Must Know  
**Difficulty:** Tricky

**What the interviewer is testing:** A common production-level mistake.

**Expected Answer:**

If the blocking function executes on the event-loop thread, it can prevent the event loop from processing other asynchronous tasks efficiently.

For example:

```python
@app.get("/bad")
async def bad_endpoint():
    time.sleep(5)
    return {"ok": True}
```

`time.sleep()` blocks the executing thread.

For blocking operations, I should either:
- Use an appropriate asynchronous library.
- Run the blocking operation in a thread pool when suitable.
- Move CPU-heavy work to worker processes/threads or a separate service.

**Key Points to Remember:**
- `async def` does not magically convert blocking functions into non-blocking operations.
- Avoid `time.sleep()` in async endpoints.
- Choose async libraries for async applications where appropriate.

---

# 6. Middleware and Application Lifecycle

### Q18. What is middleware in FastAPI?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Understanding of request processing outside route handlers.

**Expected Answer:**

Middleware is code that runs around request processing. It can inspect or modify requests and responses and perform cross-cutting operations.

Typical use cases include:
- Logging.
- CORS.
- Authentication-related processing.
- Request timing.
- Headers.
- Metrics.

Conceptually:

```text
Request
  ↓
Middleware
  ↓
Route
  ↓
Middleware
  ↓
Response
```

**Key Points to Remember:**
- Middleware can run before and after route processing.
- It applies broadly to requests.
- Use it for cross-cutting concerns rather than endpoint-specific business logic.

---

### Q19. What is CORS and how is it configured in FastAPI?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Practical knowledge for frontend/backend integration.

**Expected Answer:**

CORS stands for Cross-Origin Resource Sharing. Browsers enforce same-origin restrictions, so a frontend hosted on one origin may need permission to call an API hosted on another origin.

FastAPI can configure CORS middleware:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://example.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

In production, I would avoid blindly allowing every origin unless the application's requirements justify it.

**Key Points to Remember:**
- CORS is primarily a browser security mechanism.
- Configure allowed origins carefully.
- CORS is not authentication.

---

### Q20. How do startup/shutdown or application lifespan events work in FastAPI?

**Priority:** 🟠 Important  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of resource initialization and cleanup.

**Expected Answer:**

FastAPI provides application lifespan mechanisms for work that should happen when the application starts and shuts down.

Typical examples include:
- Initializing resources.
- Loading a model.
- Creating shared clients.
- Releasing resources during shutdown.

Modern FastAPI applications can use the lifespan mechanism to manage setup and cleanup.

Conceptually:

```text
Application starts
      ↓
Initialize resources
      ↓
Serve requests
      ↓
Application shuts down
      ↓
Release resources
```

**Key Points to Remember:**
- Use lifecycle management for application-wide resources.
- Clean up resources properly.
- Prefer current lifespan patterns rather than relying on deprecated approaches.

---

# 7. Responses, Errors, and Background Work

### Q21. How do you define response models and HTTP status codes in FastAPI?

**Priority:** 🟠 Important  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you can build clean API contracts.

**Expected Answer:**

FastAPI allows me to specify response models and status codes.

```python
@app.post(
    "/users",
    response_model=UserResponse,
    status_code=201
)
def create_user(user: UserCreate):
    ...
```

The response model documents and validates/serializes the intended response structure, while the status code communicates the result of the HTTP operation.

I can also raise explicit HTTP errors:

```python
raise HTTPException(
    status_code=404,
    detail="User not found"
)
```

**Key Points to Remember:**
- Use meaningful HTTP status codes.
- Response models make API contracts explicit.
- Don't expose internal exceptions directly to clients.

---

### Q22. What are BackgroundTasks in FastAPI, and when would you use them?

**Priority:** 🟡 Good to Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you understand lightweight background work versus real job queues.

**Expected Answer:**

FastAPI provides `BackgroundTasks` for running small tasks after returning a response.

Examples can include:
- Writing a small audit record.
- Sending a simple notification.
- Performing lightweight cleanup.

However, for critical, long-running, CPU-heavy, or highly reliable background jobs, I would generally use a dedicated task queue/worker system such as Celery or another appropriate architecture.

**Key Points to Remember:**
- Useful for lightweight background work.
- Not a replacement for a distributed job queue.
- Reliability and retry requirements should influence the architecture.

---

# 8. Testing, Performance, and Security

### Q23. How do you test a FastAPI application?

**Priority:** 🟠 Important  
**Difficulty:** Intermediate

**What the interviewer is testing:** Practical development and testing knowledge.

**Expected Answer:**

FastAPI integrates well with Python testing tools such as `pytest`. A test client can send requests to the application without requiring a real external browser.

A typical test checks:
- Status code.
- Response body.
- Validation behavior.
- Authentication.
- Error cases.

Example:

```python
def test_get_user(client):
    response = client.get("/users/1")

    assert response.status_code == 200
```

For database-backed applications, I would isolate test data and dependencies so tests remain deterministic.

**Key Points to Remember:**
- Test successful and failure paths.
- Test validation and authentication.
- Mock or override external dependencies when appropriate.
- Keep tests independent.

---

### Q24. How would you improve the performance of a FastAPI application?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Practical performance reasoning.

**Expected Answer:**

I would first measure where the bottleneck is rather than optimizing blindly.

Potential improvements include:
- Use asynchronous I/O for suitable workloads.
- Avoid blocking the event loop.
- Optimize database queries and indexes.
- Use connection pooling.
- Add caching where appropriate.
- Paginate large responses.
- Avoid unnecessary serialization.
- Stream large responses when appropriate.
- Run multiple worker processes when the deployment architecture requires it.
- Move CPU-heavy tasks to workers or separate services.

**Key Points to Remember:**
- Measure first.
- Database performance is often more important than framework-level micro-optimizations.
- Async helps I/O concurrency, not CPU computation.
- Scaling workers is different from making a single event loop faster.

---

### Q25. What are common FastAPI mistakes and security best practices?

**Priority:** 🔴 Must Know  
**Difficulty:** Tricky

**What the interviewer is testing:** Whether you can safely use FastAPI in a real application.

**Expected Answer:**

Common mistakes include:

1. Trusting client input without validation.
2. Hardcoding API keys, database passwords, or JWT secrets.
3. Returning sensitive fields in API responses.
4. Using overly permissive CORS settings.
5. Putting blocking operations inside async endpoints.
6. Not implementing authentication/authorization correctly.
7. Exposing detailed internal exceptions or stack traces.
8. Not limiting large request bodies or uploads.
9. Forgetting rate limiting for sensitive endpoints.
10. Not validating uploaded files and user-controlled data.

For production, I would use environment/secret management, Pydantic validation, secure authentication, least-privilege authorization, HTTPS, careful CORS configuration, structured logging, dependency updates, request limits, and proper error handling.

**Key Points to Remember:**
- Validation is necessary but is not the same as authorization.
- Authentication answers "Who are you?"
- Authorization answers "What are you allowed to do?"
- Never expose secrets or sensitive internal details.

---

# Final Revision Section

## 1. 🔴 Must-Know Questions

You should be able to answer these without hesitation:

1. What is FastAPI?
2. Why choose FastAPI over Flask/Django?
3. What are FastAPI's main components?
4. What is ASGI?
5. FastAPI vs Uvicorn?
6. How are routes defined?
7. Path vs query parameters?
8. How are request bodies handled?
9. How does automatic documentation work?
10. What is Pydantic?
11. How does request validation work?
12. What is dependency injection?
13. How can dependencies be used for authentication?
14. `def` vs `async def`?
15. Does `async def` automatically make an application faster?
16. What happens if blocking code runs inside an async endpoint?
17. What is middleware?
18. What is CORS?
19. How do response models work?
20. How would you improve FastAPI performance?
21. What security practices would you follow?

---

# 2. ⚠️ Tricky Concepts

### FastAPI ≠ Uvicorn

FastAPI is the application framework. Uvicorn is an ASGI server that runs the application.

### Async ≠ automatically faster

`async` is useful when the application spends time waiting for asynchronous I/O. It does not make CPU-bound calculations faster.

### `async def` + blocking code is still blocking

This is a critical misconception.

```python
async def endpoint():
    time.sleep(10)  # blocks
```

The keyword `async` does not transform `time.sleep()` into asynchronous I/O.

### ASGI ≠ FastAPI

ASGI is an interface/specification. FastAPI is a framework that implements an ASGI application.

### Validation ≠ authentication

Pydantic can tell you whether data has the expected shape/type.

It cannot tell you whether the user is allowed to perform an operation.

### Authentication ≠ authorization

```text
Authentication → Who are you?
Authorization  → What can you do?
```

### BackgroundTasks ≠ distributed job queue

FastAPI's lightweight background task mechanism should not automatically be treated as a replacement for Celery, a message queue, or a dedicated worker architecture.

---

# 3. 🧠 Quick Revision Sheet

## FastAPI Request Flow

```text
Client
  ↓
Uvicorn / ASGI Server
  ↓
FastAPI
  ↓
Middleware
  ↓
Dependency Resolution
  ↓
Request Validation
  ↓
Route Handler
  ↓
Business Logic
  ↓
Response Model / Serialization
  ↓
HTTP Response
```

## Main Technologies

| Technology | Role |
|---|---|
| FastAPI | Python API framework |
| Starlette | Web/ASGI foundation |
| Pydantic | Validation and data models |
| Uvicorn | ASGI server |
| OpenAPI | API specification |
| Swagger UI | Interactive API docs |
| ReDoc | Alternative API documentation UI |
| pytest | Testing framework |

## Core FastAPI Features

```text
FastAPI
 ├── Routing
 ├── Type hints
 ├── Request validation
 ├── Response models
 ├── Dependency injection
 ├── Async support
 ├── Middleware
 ├── Authentication utilities
 ├── OpenAPI generation
 └── Interactive documentation
```

---

# 4. 🔑 Important Terms

Be ready to recognize and explain:

- FastAPI
- ASGI
- WSGI
- Uvicorn
- Starlette
- Pydantic
- OpenAPI
- Swagger UI
- ReDoc
- Route
- Path parameter
- Query parameter
- Request body
- Request model
- Response model
- Serialization
- Deserialization/parsing
- Validation
- Dependency injection
- `Depends`
- Middleware
- CORS
- Authentication
- Authorization
- OAuth2
- JWT
- Bearer token
- Async
- Await
- Coroutine
- Event loop
- Thread pool
- Worker
- Lifespan
- Background task
- HTTPException
- Status code
- API schema
- Dependency override
- Connection pooling

---

# 5. 🎯 Interview Tips

### 1. Know the FastAPI stack

If asked:

> "How does FastAPI work?"

A strong fresher answer is:

> "FastAPI is an API framework built on Starlette and Pydantic. It exposes an ASGI application, which can be served by an ASGI server such as Uvicorn. Starlette handles the web layer, while Pydantic handles structured data validation and serialization. FastAPI connects these pieces with type-driven routing, dependency injection, and OpenAPI generation."

That is much stronger than simply saying "FastAPI is a Python framework."

### 2. Be able to explain one complete request

For example:

```text
POST /users
      ↓
Uvicorn
      ↓
FastAPI route matching
      ↓
Dependency resolution
      ↓
Pydantic validates JSON
      ↓
Endpoint executes
      ↓
Database operation
      ↓
Response model
      ↓
JSON response
```

### 3. If FastAPI is on your resume, expect project-specific follow-ups

Be prepared to explain:

- Why you selected FastAPI.
- How your routes were organized.
- How your Pydantic models were designed.
- How validation worked.
- How your frontend communicated with FastAPI.
- How CORS was configured.
- How authentication was implemented.
- How database connections were managed.
- Whether endpoints were synchronous or asynchronous.
- How errors were handled.
- How the application was deployed.
- How you tested your APIs.

### 4. Don't claim async expertise just because you wrote `async def`

An interviewer may immediately ask:

> "What happens if you call a blocking function inside it?"

You should know the answer.

### 5. Know why Pydantic is valuable

A strong explanation is:

> "Pydantic lets me define explicit data contracts using Python type annotations. FastAPI uses those models to validate input, parse data, serialize responses, and generate API documentation."

---

# 6. ❓ Self-Test

Try answering these aloud without looking at the answers:

1. Explain FastAPI in 30 seconds.
2. Why would you choose FastAPI instead of Flask?
3. FastAPI vs Django?
4. What is Starlette?
5. What is Pydantic?
6. What is ASGI?
7. What is Uvicorn?
8. FastAPI vs Uvicorn?
9. Explain the lifecycle of a FastAPI request.
10. How does FastAPI determine whether a parameter is a path or query parameter?
11. How does request-body validation work?
12. What happens when validation fails?
13. How does FastAPI generate Swagger documentation?
14. What is dependency injection?
15. How would you use `Depends()` for authentication?
16. `def` vs `async def` in FastAPI?
17. Does `async def` automatically make an endpoint faster?
18. What happens if `time.sleep()` is used in an async endpoint?
19. What is middleware?
20. What is CORS?
21. Authentication vs authorization?
22. Request model vs response model?
23. How would you handle errors?
24. How would you improve FastAPI performance?
25. What security practices would you implement before deploying a FastAPI application?

---

# Final Interview Target

For a fresher Software Engineering interview, answer FastAPI questions using:

> **Framework → underlying technology → mechanism → practical benefit → limitation**

For example:

> "FastAPI is a Python framework for building APIs. It is built on Starlette for the web/ASGI layer and uses Pydantic for validation and data models. Its type-hint-based design allows it to automatically validate requests and generate OpenAPI documentation. It also supports async endpoints, which are useful for I/O-bound workloads. However, using `async def` does not make blocking or CPU-heavy code asynchronous, so those workloads need to be handled differently."

That answer demonstrates **framework knowledge, internal understanding, practical usage, and trade-off awareness**—the level expected when FastAPI appears on a fresher Software Engineering resume.
