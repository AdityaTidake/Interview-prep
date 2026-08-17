# Node.js — Fresher Software Engineering Interview Revision Guide

> **Goal:** Be able to confidently defend Node.js as a technology on your resume, including its runtime model, event loop, asynchronous programming, modules, HTTP servers, performance, security, and common pitfalls.

---

# 1. Fundamentals

### Q1. What is Node.js?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Whether you understand what Node.js actually is.

**Expected Answer:**

Node.js is a JavaScript runtime environment that allows JavaScript to execute outside the browser. It is built on Google's V8 JavaScript engine and provides APIs for things such as networking, files, processes, and streams.

Node.js is particularly useful for I/O-heavy applications because its event-driven architecture allows it to handle many concurrent I/O operations without creating one blocking thread per request.

**Key Points to Remember:**
- Node.js is a **runtime**, not a programming language.
- It uses the V8 JavaScript engine.
- JavaScript can run on the server.
- Strong fit for I/O-heavy and network applications.
- It does not mean CPU-heavy work automatically becomes fast.

---

### Q2. Why is Node.js popular for backend development?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Whether you understand its practical advantages.

**Expected Answer:**

Node.js is popular because it provides an asynchronous, event-driven programming model that works well for applications with many I/O operations, such as APIs, real-time applications, and network services.

Another advantage is that the same JavaScript/TypeScript ecosystem can be used across frontend and backend. Node.js also has a large package ecosystem through npm.

**Key Points to Remember:**
- Efficient I/O concurrency.
- Large npm ecosystem.
- Same language across frontend/backend.
- Good support for HTTP, WebSockets, streams, and APIs.
- Not automatically the best choice for every workload.

---

### Q3. What is V8?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Whether you know the JavaScript engine behind Node.js.

**Expected Answer:**

V8 is Google's open-source JavaScript and WebAssembly engine, originally developed for Chrome. Node.js uses V8 to parse, compile, and execute JavaScript.

V8 includes mechanisms such as JIT compilation and garbage collection.

A useful distinction is:

```text
Node.js
 ├── V8 → executes JavaScript
 ├── Node APIs → filesystem, networking, etc.
 └── Event loop / async infrastructure
```

**Key Points to Remember:**
- V8 executes JavaScript.
- Node.js provides runtime capabilities around the engine.
- V8 is also used by Chromium-based browsers.

---

### Q4. Is Node.js single-threaded?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** One of the most common Node.js misconceptions.

**Expected Answer:**

Node.js executes JavaScript callbacks primarily on a single main thread, but saying "Node.js is completely single-threaded" is inaccurate.

Node.js can use other threads internally for certain operations through its runtime infrastructure, and applications can explicitly use mechanisms such as `worker_threads` for CPU-intensive JavaScript work.

The key idea is that normal JavaScript execution is coordinated through the event loop rather than creating a dedicated application thread for every request.

**Key Points to Remember:**
- JavaScript execution is primarily on one main thread.
- Node.js can use background threads internally.
- `worker_threads` can run JavaScript in separate threads.
- Single-threaded JavaScript does not mean only one thread exists anywhere in Node.js.

---

### Q5. What is npm?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Familiarity with the Node.js package ecosystem.

**Expected Answer:**

npm is the package manager and registry ecosystem commonly used with Node.js. It allows developers to install, update, and manage project dependencies and run scripts defined in `package.json`.

For example:

```bash
npm install express
npm run dev
```

A Node.js project commonly has a `package.json` file describing metadata, scripts, and dependencies.

**Key Points to Remember:**
- npm manages packages and dependencies.
- `package.json` defines project configuration.
- npm scripts automate common commands.
- `package-lock.json` helps lock dependency versions.

---

# 2. Node.js Architecture and Event Loop

### Q6. Explain the Node.js architecture.

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you understand how Node.js handles requests and asynchronous work.

**Expected Answer:**

A simplified Node.js architecture looks like:

```text
Application JavaScript
        ↓
Node.js APIs
        ↓
Event loop + async infrastructure
        ↓
OS / libuv / worker pool
        ↓
Network, filesystem, timers, etc.
```

Node.js runs JavaScript on the main thread. When asynchronous I/O is initiated, Node.js can delegate the underlying operation to the operating system or libuv's worker pool, depending on the operation.

When the operation completes, its callback or promise continuation is made available for execution by the event loop.

**Key Points to Remember:**
- V8 executes JavaScript.
- Node APIs expose runtime functionality.
- libuv provides important asynchronous infrastructure.
- The event loop coordinates callback execution.
- Not every asynchronous operation uses the worker pool.

---

### Q7. What is the event loop in Node.js?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Core Node.js asynchronous behavior.

**Expected Answer:**

The event loop is the mechanism that allows Node.js to execute JavaScript callbacks and coordinate asynchronous operations without blocking the main JavaScript thread while I/O is waiting.

A simplified model is:

```text
Start operation
     ↓
Async I/O / timer / etc.
     ↓
Node continues executing other work
     ↓
Operation completes
     ↓
Callback / promise continuation becomes ready
     ↓
Event loop executes it
```

Node's event loop has multiple phases, including timers, pending callbacks, poll, check, and close callbacks. The exact scheduling behavior depends on the operation and Node.js version.

**Key Points to Remember:**
- Event loop enables asynchronous concurrency.
- It does not execute multiple JavaScript statements simultaneously on the same main thread.
- Blocking the event loop hurts all requests handled by that process.

---

### Q8. What is libuv and why is it important?

**Priority:** 🟠 Important  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you understand what sits beneath Node's high-level APIs.

**Expected Answer:**

libuv is a cross-platform C library used by Node.js to provide important asynchronous I/O infrastructure and the event loop.

It helps Node.js interact with the operating system for operations such as networking and filesystem work and provides a thread pool for certain operations that should not block the main JavaScript thread.

**Key Points to Remember:**
- libuv is implemented in C.
- It provides the event loop infrastructure.
- It supports cross-platform asynchronous operations.
- Its worker pool handles certain operations.
- Do not say "all async work goes to the thread pool."

---

### Q9. What are the phases of the Node.js event loop?

**Priority:** 🟠 Important  
**Difficulty:** Tricky

**What the interviewer is testing:** Whether you understand the event loop beyond a basic definition.

**Expected Answer:**

A simplified view of Node.js event-loop phases is:

```text
timers
   ↓
pending callbacks
   ↓
poll
   ↓
check
   ↓
close callbacks
```

There are additional details and version-specific scheduling behavior, especially around timers and microtasks.

- **Timers:** Executes callbacks for eligible timers.
- **Poll:** Handles I/O events and related callbacks.
- **Check:** Executes `setImmediate()` callbacks.
- **Close callbacks:** Handles certain close events.

`process.nextTick()` and promise microtasks have special scheduling behavior and should not simply be treated as another event-loop phase.

**Key Points to Remember:**
- Know timers, poll, check, and close callbacks.
- `setImmediate()` is associated with the check phase.
- Microtasks have priority behavior that differs from ordinary event-loop phases.

---

### Q10. What is the difference between `process.nextTick()`, Promise callbacks, `setTimeout()`, and `setImmediate()`?

**Priority:** 🔴 Must Know  
**Difficulty:** Tricky

**What the interviewer is testing:** Whether you genuinely understand asynchronous scheduling.

**Expected Answer:**

They all schedule work differently.

- `process.nextTick()` schedules a callback to run after the current operation, before the event loop continues normally.
- Promise callbacks such as `.then()` are microtasks.
- `setTimeout()` schedules a timer callback after its delay becomes eligible.
- `setImmediate()` schedules a callback for the event loop's check phase.

For example:

```js
console.log("A");

setTimeout(() => console.log("timeout"), 0);

setImmediate(() => console.log("immediate"));

Promise.resolve().then(() => console.log("promise"));

process.nextTick(() => console.log("nextTick"));

console.log("B");
```

The exact relative ordering of `setTimeout(0)` and `setImmediate()` can depend on context, especially whether the code runs inside an I/O callback.

**Key Points to Remember:**
- `nextTick` and Promise callbacks are not ordinary timer callbacks.
- Do not claim `setTimeout(0)` always executes before `setImmediate()`.
- Excessive `nextTick` usage can starve the event loop.

---

# 3. Asynchronous Programming

### Q11. What does non-blocking I/O mean in Node.js?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Understanding of Node's core execution model.

**Expected Answer:**

Non-blocking I/O means Node.js can start an I/O operation and continue executing other JavaScript instead of waiting synchronously for that operation to finish.

For example:

```js
fs.readFile("data.txt", (err, data) => {
  console.log(data);
});

console.log("This can execute before the file read callback");
```

The JavaScript thread is not sitting idle waiting for the file operation.

**Key Points to Remember:**
- Non-blocking does not mean "instant."
- It means the JavaScript thread does not synchronously wait for the I/O.
- This helps Node handle many concurrent I/O operations efficiently.

---

### Q12. What are callbacks, Promises, and `async/await` in Node.js?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Knowledge of Node's asynchronous programming styles.

**Expected Answer:**

They are different ways of handling asynchronous operations.

**Callback:**

```js
fs.readFile("a.txt", (err, data) => {
  // handle result
});
```

**Promise:**

```js
readFilePromise("a.txt")
  .then(data => ...)
  .catch(err => ...);
```

**async/await:**

```js
try {
  const data = await readFilePromise("a.txt");
} catch (err) {
  // handle error
}
```

`async/await` is syntax built around Promises and often makes asynchronous control flow easier to read.

**Key Points to Remember:**
- `async` functions return Promises.
- `await` pauses that async function until the Promise settles; it does not block the Node.js process.
- Always handle rejection/errors appropriately.

---

### Q13. Does `await` block Node.js?

**Priority:** 🔴 Must Know  
**Difficulty:** Tricky

**What the interviewer is testing:** A common misconception.

**Expected Answer:**

`await` suspends the execution of the current `async` function until the awaited Promise settles, but it does not block the Node.js event loop in the same way as synchronous blocking code.

For example:

```js
const data = await fetchData();
```

While the operation is pending, Node can continue processing other work.

However, if the awaited operation itself performs CPU-heavy synchronous work, the event loop can still be blocked.

**Key Points to Remember:**
- `await` pauses the current async function.
- It does not freeze the entire Node.js process.
- CPU-heavy synchronous work can still block the event loop.

---

### Q14. What is callback hell, and how can it be avoided?

**Priority:** 🟠 Important  
**Difficulty:** Basic

**What the interviewer is testing:** Understanding of asynchronous code structure.

**Expected Answer:**

Callback hell occurs when deeply nested callbacks make asynchronous code difficult to read, maintain, and handle errors in.

```text
callback
  └── callback
       └── callback
            └── callback
```

It can be reduced using:
- Promises.
- `async/await`.
- Smaller functions.
- Proper error handling.
- Appropriate abstractions.

**Key Points to Remember:**
- It is mainly a readability/maintainability problem.
- Promises and `async/await` make sequential async flows easier to express.

---

# 4. Modules and Node.js APIs

### Q15. What is the difference between CommonJS and ES Modules?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of Node.js module systems.

**Expected Answer:**

Node.js supports both CommonJS and ECMAScript Modules.

**CommonJS:**

```js
const express = require("express");
module.exports = router;
```

**ES Modules:**

```js
import express from "express";
export default router;
```

ES Modules are the standard JavaScript module system. Node determines how files are interpreted based on configuration such as the `type` field in `package.json` and file extensions.

**Key Points to Remember:**
- CommonJS → `require`, `module.exports`.
- ESM → `import`, `export`.
- Know which module system your project uses.
- Avoid mixing systems without understanding interoperability rules.

---

### Q16. What are Node.js built-in modules you commonly use?

**Priority:** 🟡 Good to Know  
**Difficulty:** Basic

**What the interviewer is testing:** Practical familiarity with the Node runtime.

**Expected Answer:**

Common built-in modules include:

- `fs` → filesystem operations.
- `path` → path manipulation.
- `http` / `https` → HTTP networking.
- `url` → URL parsing/manipulation.
- `events` → EventEmitter functionality.
- `crypto` → cryptographic functionality.
- `stream` → streaming data.
- `os` → operating-system information.
- `process` → process/environment information.

Example:

```js
import path from "node:path";

const filePath = path.join("data", "users.json");
```

**Key Points to Remember:**
- Built-in modules don't require npm installation.
- Prefer the `node:` prefix in modern Node code when appropriate.

---

### Q17. What is `EventEmitter` in Node.js?

**Priority:** 🟠 Important  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of Node's event-driven programming model.

**Expected Answer:**

`EventEmitter` provides a mechanism for objects to emit named events and for other code to register listeners for those events.

```js
import { EventEmitter } from "node:events";

const emitter = new EventEmitter();

emitter.on("login", user => {
  console.log(user);
});

emitter.emit("login", { id: 1 });
```

It is used internally by several Node.js APIs and can also be useful for application-level event handling.

**Key Points to Remember:**
- `on()` registers a listener.
- `emit()` triggers an event.
- Event-driven design can decouple producers and consumers.
- Uncontrolled listeners can create memory/performance problems.

---

# 5. HTTP and Backend Development

### Q18. How do you create a basic HTTP server in Node.js?

**Priority:** 🟠 Important  
**Difficulty:** Basic

**What the interviewer is testing:** Whether you understand Node's HTTP foundation.

**Expected Answer:**

Node provides the built-in `http` module:

```js
import http from "node:http";

const server = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "application/json" });
  res.end(JSON.stringify({ message: "Hello" }));
});

server.listen(3000);
```

The request handler receives the incoming request and response objects.

Frameworks such as Express build higher-level routing and middleware abstractions on top of Node's HTTP capabilities.

**Key Points to Remember:**
- Node can create HTTP servers without Express.
- Frameworks provide abstractions.
- Understand the underlying `req` and `res` concepts.

---

### Q19. What is middleware in a Node.js framework such as Express?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Practical backend knowledge.

**Expected Answer:**

Middleware is a function that participates in processing an HTTP request before the final response is sent. It can inspect or modify the request/response, perform authentication, logging, validation, parsing, and either pass control onward or terminate the request.

```js
app.use((req, res, next) => {
  console.log(req.method, req.url);
  next();
});
```

`next()` passes control to the next middleware or route handler.

**Key Points to Remember:**
- Middleware executes in an order.
- It can modify `req`/`res`.
- It can terminate the request.
- Authentication, validation, logging, and error handling are common middleware use cases.

---

### Q20. How does error handling work in Node.js backend applications?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you can build reliable Node.js services.

**Expected Answer:**

Error handling depends on the API and programming style.

For Promise-based code, use `try/catch` around awaited operations or handle Promise rejections.

```js
try {
  const user = await getUser(id);
} catch (err) {
  // handle/log error
}
```

In Express-style applications, centralized error middleware is commonly used:

```js
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: "Internal server error" });
});
```

Errors should be logged with enough context for debugging without exposing sensitive information to clients.

**Key Points to Remember:**
- Don't leave Promise rejections unhandled.
- Centralize HTTP error formatting.
- Don't expose stack traces or secrets in production responses.
- Distinguish operational errors from programming bugs where appropriate.

---

# 6. Streams, Buffers, and Performance

### Q21. What are streams in Node.js?

**Priority:** 🟠 Important  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of efficient data processing.

**Expected Answer:**

Streams allow data to be processed incrementally rather than loading the entire dataset into memory at once.

For example, a large file can be read as chunks:

```text
Large file
   ↓
Chunk 1 → process
Chunk 2 → process
Chunk 3 → process
...
```

Node provides readable, writable, duplex, and transform streams.

Streams are useful for files, HTTP requests/responses, compression, and other continuous data flows.

**Key Points to Remember:**
- Streams reduce memory pressure for large data.
- Data is processed incrementally.
- Understand backpressure.
- Common types: Readable, Writable, Duplex, Transform.

---

### Q22. What is a Buffer in Node.js?

**Priority:** 🟡 Good to Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of binary data handling.

**Expected Answer:**

A Buffer is a Node.js representation for a sequence of bytes. It is useful when working with binary data such as files, network packets, images, and encoded data.

```js
const buffer = Buffer.from("Hello");
console.log(buffer);
```

Buffers are commonly used with streams and network/file APIs.

**Key Points to Remember:**
- Buffer represents raw bytes.
- Useful for binary data.
- It is different from a normal JavaScript string.

---

### Q23. What is backpressure in Node.js streams?

**Priority:** 🟠 Important  
**Difficulty:** Tricky

**What the interviewer is testing:** Deeper understanding of stream performance.

**Expected Answer:**

Backpressure occurs when a producer generates data faster than a consumer can process it.

For example:

```text
Fast producer
     ↓↓↓↓↓↓↓
Slow consumer
     ↓
Buffer fills
     ↓
Memory pressure / instability
```

Node streams provide mechanisms to manage this flow so the producer does not continuously overwhelm the consumer.

Using `pipe()` is one common way to connect streams while allowing stream backpressure handling.

**Key Points to Remember:**
- Producer faster than consumer → backpressure.
- Ignoring backpressure can increase memory usage.
- Streams are useful partly because they provide controlled data flow.

---

# 7. CPU, Concurrency, and Scaling

### Q24. What happens if you perform CPU-intensive work in a Node.js request handler?

**Priority:** 🔴 Must Know  
**Difficulty:** Tricky

**What the interviewer is testing:** Whether you understand the main limitation of Node's event-driven model.

**Expected Answer:**

If CPU-intensive JavaScript runs synchronously on the main thread, it blocks the event loop.

For example:

```js
app.get("/calculate", (req, res) => {
  const result = expensiveCalculation();
  res.json({ result });
});
```

While `expensiveCalculation()` is running, that Node.js process cannot execute other JavaScript callbacks on the same main thread.

Possible solutions include:
- `worker_threads` for CPU-intensive JavaScript work.
- Separate worker processes/services.
- Moving computation to another service.
- Using asynchronous native operations when appropriate.

**Key Points to Remember:**
- Node is excellent for I/O-heavy workloads.
- CPU-heavy synchronous work can block all requests handled by that event loop.
- Async syntax alone does not make CPU-bound work non-blocking.

---

### Q25. What are common Node.js mistakes and best practices?

**Priority:** 🔴 Must Know  
**Difficulty:** Tricky

**What the interviewer is testing:** Whether your practical Node.js knowledge is strong enough to defend the technology on your resume.

**Expected Answer:**

Important mistakes to avoid include:

1. Blocking the event loop with synchronous CPU-heavy operations.
2. Ignoring Promise rejections.
3. Not validating user input.
4. Hardcoding secrets instead of using environment/configuration management.
5. Returning overly detailed internal errors.
6. Not implementing request timeouts or appropriate limits.
7. Loading huge files entirely into memory when streams would be better.
8. Creating unbounded concurrency with `Promise.all()` for huge workloads.
9. Not handling graceful shutdown.
10. Using outdated or vulnerable dependencies without monitoring them.

For production applications, I would also consider structured logging, authentication/authorization, rate limiting, input validation, dependency updates, health checks, and appropriate process/container management.

**Key Points to Remember:**
- Protect the event loop.
- Validate inputs.
- Handle errors and rejected Promises.
- Protect secrets.
- Use streams for large data.
- Control concurrency.
- Monitor dependencies and application health.

---

# Final Revision Section

## 1. 🔴 Must-Know Questions

You should be able to answer these without hesitation:

1. What is Node.js?
2. Why is Node.js useful for backend development?
3. What is V8?
4. Is Node.js single-threaded?
5. Explain Node.js architecture.
6. What is the event loop?
7. What does non-blocking I/O mean?
8. Explain callbacks, Promises, and `async/await`.
9. Does `await` block Node.js?
10. CommonJS vs ES Modules.
11. What is middleware?
12. How does error handling work?
13. What happens when CPU-heavy code runs on the main thread?
14. What are streams?
15. What is backpressure?
16. What are common Node.js mistakes?

---

# 2. ⚠️ Tricky Concepts

### Node.js is not "single-threaded everywhere"

JavaScript execution is primarily on the main thread, but Node can use other threads internally and applications can use worker threads.

### Asynchronous does not mean parallel

Node can coordinate many pending I/O operations, but ordinary JavaScript execution on the main thread is not simultaneously running on multiple CPU cores.

### `await` does not block the event loop

It suspends the current async function while the Promise is pending.

### CPU-heavy code can still block Node

This is one of the most important concepts.

```text
I/O-heavy:
Request → async I/O → other work → callback
                     ↑
               event loop remains available

CPU-heavy:
Request → huge synchronous calculation → event loop blocked
```

### `setTimeout(0)` does not mean "execute immediately"

It means the callback becomes eligible after the timer threshold and event-loop scheduling permits it.

### Not every asynchronous operation uses the worker pool

Some operations are handled through OS facilities or Node's networking/event infrastructure.

### Streams are not only about speed

They also reduce memory usage and support controlled data flow through backpressure.

---

# 3. 🧠 Quick Revision Sheet

## Node.js Mental Model

```text
                 Node.js Application
                         │
                         ▼
                    JavaScript
                         │
                         ▼
                        V8
                         │
          ┌──────────────┴──────────────┐
          ▼                             ▼
     Node.js APIs                  Event Loop
          │                             │
          └──────────────┬──────────────┘
                         ▼
                  OS / libuv / pool
                         │
                         ▼
                Async I/O operations
                         │
                         ▼
                 Callback / Promise
                         │
                         ▼
                    Event Loop
```

## Core Components

| Component | Role |
|---|---|
| V8 | Executes JavaScript |
| Node APIs | Provide runtime capabilities |
| libuv | Async I/O and event-loop infrastructure |
| Event loop | Coordinates callback execution |
| Worker pool | Handles certain operations away from main JS execution |
| npm | Package management ecosystem |
| `package.json` | Project metadata, dependencies, scripts |
| Streams | Incremental data processing |
| Buffer | Binary byte data |

## Async Model

```text
Synchronous code
      ↓
Runs immediately on JS thread

Async I/O
      ↓
Start operation
      ↓
Continue other JS
      ↓
Operation completes
      ↓
Callback / Promise continuation
      ↓
Event loop
```

---

# 4. 🔑 Important Terms

Be ready to recognize and explain:

- Node.js
- Runtime
- V8
- libuv
- Event loop
- Event-driven architecture
- Non-blocking I/O
- Asynchronous I/O
- Callback
- Promise
- `async/await`
- Microtask
- `process.nextTick`
- `setImmediate`
- `setTimeout`
- CommonJS
- ES Modules
- npm
- `package.json`
- `package-lock.json`
- EventEmitter
- Middleware
- HTTP server
- Request/response
- Stream
- Readable stream
- Writable stream
- Duplex stream
- Transform stream
- Buffer
- Backpressure
- Worker thread
- Child process
- Process
- Graceful shutdown
- Environment variables
- Rate limiting

---

# 5. 🎯 Interview Tips

### 1. Be able to draw the event-loop architecture

If an interviewer asks "How does Node.js work?", don't give only a definition.

Draw:

```text
Request
   ↓
Node.js
   ↓
Event Loop
   ↓
Async operation
   ↓
OS / libuv
   ↓
Completion
   ↓
Callback / Promise
   ↓
Response
```

Then explain where JavaScript executes and where asynchronous work happens.

### 2. Don't say "Node.js creates a thread for every request"

That is not the Node.js model.

The key design is an event-driven model where many requests can share the same JavaScript execution thread while I/O is pending.

### 3. Be prepared to explain your actual Node.js project

If Node.js is on your resume, expect:

- Why did you choose Node.js?
- Which framework did you use?
- How did your API routes work?
- How did middleware work?
- How did you handle errors?
- How did you validate requests?
- How did you connect to the database?
- How did you handle authentication?
- How did you structure your project?
- How did you handle asynchronous operations?
- Did you encounter any performance issues?
- How would you scale multiple Node.js instances?

### 4. Don't confuse Express with Node.js

A strong answer:

> "Node.js provides the runtime and low-level HTTP/runtime APIs. Express is a framework that provides routing, middleware, and other abstractions that make HTTP application development easier."

### 5. Know the limitation

A very good fresher answer is:

> "Node.js is particularly strong for I/O-bound workloads, but CPU-heavy synchronous JavaScript can block the event loop, so CPU-intensive tasks may need worker threads or separate services."

That single point often leads to useful follow-up questions.

---

# 6. ❓ Self-Test

Try answering these aloud without looking at the answers:

1. Explain Node.js in 30 seconds.
2. Node.js vs JavaScript?
3. Node.js vs browser JavaScript?
4. What is V8?
5. What is libuv?
6. Explain the Node.js event loop.
7. Why is Node.js called event-driven?
8. What does non-blocking I/O mean?
9. Is Node.js single-threaded?
10. What happens when a Node.js application receives 1,000 simultaneous I/O requests?
11. What happens if one request performs a 10-second CPU-heavy synchronous calculation?
12. Does `await` block the event loop?
13. `process.nextTick()` vs Promise microtasks?
14. `setTimeout()` vs `setImmediate()`?
15. Callback vs Promise vs `async/await`?
16. CommonJS vs ES Modules?
17. What is npm?
18. What is middleware?
19. What is EventEmitter?
20. What are streams?
21. What is backpressure?
22. What is a Buffer?
23. How would you handle errors in an Express application?
24. How would you handle CPU-intensive work?
25. What Node.js performance/security best practices have you actually used?

---

# Final Interview Target

For a fresher Software Engineering interview, use this structure when answering Node.js questions:

> **What it is → How it works → Why it is useful → Limitation/trade-off**

For example:

> "Node.js is a JavaScript runtime built on V8. It uses an event-driven architecture and non-blocking I/O, allowing a process to handle many concurrent I/O operations without blocking the JavaScript thread while those operations are waiting. It's a strong choice for APIs and real-time applications, but CPU-heavy synchronous work can block the event loop, so such work may need worker threads or separate services."

That style demonstrates that you understand **both the technology and its trade-offs**, rather than simply memorizing definitions.
