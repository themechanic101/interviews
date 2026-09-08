# Express.js Interview Guide --- SDE1 / SDE2

A practical Express.js interview handbook covering **basic → advanced**
concepts, with code examples, common traps, API design, middleware,
security, performance, testing, and SDE2 production scenarios.

> **Interview strategy:** Answer with **definition → how it works →
> example → production use/trade-off**. For SDE2 questions, discuss
> failure modes, scalability, security, observability, and
> maintainability.

------------------------------------------------------------------------

# 1. Express.js Fundamentals

## 1.1 What is Express.js?

Express.js is a lightweight web framework for Node.js used to build HTTP
servers, REST APIs, and web applications.

It provides abstractions for:

-   Routing
-   Middleware
-   Request/response handling
-   Error handling
-   HTTP application structure

Example:

``` js
const express = require("express");

const app = express();

app.get("/hello", (req, res) => {
  res.json({ message: "Hello Express" });
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

### Interview answer

> "Express is a minimal Node.js web framework that simplifies HTTP
> routing, middleware composition, request handling, and error handling.
> It doesn't replace Node's HTTP runtime; it builds abstractions on top
> of it."

------------------------------------------------------------------------

# 2. Express vs Node.js

## 2.1 What is the difference between Node.js and Express.js?

**Node.js** is the runtime.

**Express.js** is a web framework that runs on Node.js.

``` text
Operating System
       ↓
Node.js runtime
       ↓
HTTP module
       ↓
Express
       ↓
Application routes/middleware
```

Node's built-in HTTP server:

``` js
const http = require("node:http");

http.createServer((req, res) => {
  res.end("Hello");
}).listen(3000);
```

Express:

``` js
const express = require("express");

const app = express();

app.get("/", (req, res) => {
  res.send("Hello");
});

app.listen(3000);
```

Express gives you convenient routing and middleware APIs so you don't
have to manually implement everything around Node's HTTP primitives.

------------------------------------------------------------------------

# 3. Creating an Express Application

## 3.1 Basic application

``` js
const express = require("express");

const app = express();

app.use(express.json());

app.get("/", (req, res) => {
  res.send("Hello World");
});

app.listen(3000);
```

`express()` creates an application object.

The application object provides methods such as:

``` js
app.use()
app.get()
app.post()
app.put()
app.patch()
app.delete()
app.listen()
```

------------------------------------------------------------------------

# 4. Middleware

## 4.1 What is middleware?

Middleware is a function that participates in processing an HTTP
request.

Typical signature:

``` js
(req, res, next) => {
  // logic
  next();
}
```

Example:

``` js
app.use((req, res, next) => {
  console.log(req.method, req.url);
  next();
});
```

Flow:

``` text
Request
   ↓
Middleware 1
   ↓
Middleware 2
   ↓
Route handler
   ↓
Response
```

### Interview answer

> "Express middleware is a function in the request-processing pipeline.
> It can inspect or modify the request/response, terminate the request,
> or pass control to the next middleware using `next()`."

------------------------------------------------------------------------

## 4.2 Why is middleware important?

Middleware is useful for cross-cutting concerns:

-   Authentication
-   Authorization
-   Logging
-   Validation
-   Rate limiting
-   CORS
-   Parsing request bodies
-   Error handling
-   Request IDs

Example:

``` js
app.use(authMiddleware);
app.use("/users", userRoutes);
```

------------------------------------------------------------------------

# 5. Middleware Execution Order

## 5.1 Does middleware order matter?

**Yes.**

Express processes middleware in the order it is registered.

``` js
app.use(first);
app.use(second);

app.get("/", handler);
```

Execution:

``` text
first
  ↓
second
  ↓
handler
```

If `first` never calls `next()` and doesn't send a response, the request
can hang.

``` js
app.use((req, res, next) => {
  console.log("This request can get stuck");
  // next() missing
});
```

------------------------------------------------------------------------

# 6. `next()` vs Sending a Response

## 6.1 What happens if middleware calls `next()`?

It passes control to the next matching middleware/route.

``` js
app.use((req, res, next) => {
  console.log("middleware");
  next();
});
```

If middleware sends a response:

``` js
app.use((req, res) => {
  res.json({ message: "Done" });
});
```

it should generally **not** call `next()` afterward.

------------------------------------------------------------------------

# 7. Application-Level Middleware

## 7.1 What is application-level middleware?

Registered on the Express application:

``` js
app.use((req, res, next) => {
  console.log("Every request");
  next();
});
```

Or:

``` js
app.get(
  "/users",
  authMiddleware,
  getUsers
);
```

------------------------------------------------------------------------

# 8. Router-Level Middleware

## 8.1 What is `express.Router()`?

A Router is a modular routing object.

Example:

``` js
const express = require("express");

const router = express.Router();

router.get("/", getUsers);
router.get("/:id", getUser);

module.exports = router;
```

Mount it:

``` js
app.use("/users", router);
```

Now:

``` text
GET /users
GET /users/123
```

are handled by that router.

------------------------------------------------------------------------

# 9. Why Use Routers?

Without routers, a large file can become:

``` text
app.js
 ├── users
 ├── orders
 ├── payments
 ├── products
 ├── auth
 └── admin
```

Better:

``` text
src/
├── app.js
├── routes/
│   ├── user.routes.js
│   ├── order.routes.js
│   └── auth.routes.js
├── controllers/
├── services/
└── repositories/
```

This improves maintainability and separation of responsibilities.

------------------------------------------------------------------------

# 10. Routing

## 10.1 Basic routes

``` js
app.get("/users", handler);

app.post("/users", handler);

app.patch("/users/:id", handler);

app.delete("/users/:id", handler);
```

HTTP method + path determine which route matches.

------------------------------------------------------------------------

# 11. Route Parameters

## 11.1 What are route parameters?

Dynamic values embedded in the URL path.

``` js
app.get("/users/:id", (req, res) => {
  console.log(req.params.id);
});
```

Request:

``` text
GET /users/42
```

Then:

``` js
req.params.id
// "42"
```

Route parameters are strings initially, so validate/convert them when
numeric semantics are required.

------------------------------------------------------------------------

# 12. Query Parameters

## 12.1 What are query parameters?

They appear after `?`.

``` text
GET /users?page=2&limit=20
```

Access:

``` js
app.get("/users", (req, res) => {
  console.log(req.query.page);
  console.log(req.query.limit);
});
```

Use query parameters commonly for:

-   Filtering
-   Sorting
-   Pagination
-   Search

------------------------------------------------------------------------

# 13. Route Params vs Query Params

  Feature      Route Parameter     Query Parameter
  ------------ ------------------- ------------------
  Example      `/users/42`         `/users?page=2`
  Access       `req.params`        `req.query`
  Common use   Identify resource   Filter/options
  Required     Often               Usually optional

------------------------------------------------------------------------

# 14. Request Body

## 14.1 How do you read JSON request bodies?

Use:

``` js
app.use(express.json());
```

Then:

``` js
app.post("/users", (req, res) => {
  console.log(req.body);

  res.json({
    received: req.body
  });
});
```

Without the JSON parser middleware, `req.body` may be undefined for JSON
requests.

------------------------------------------------------------------------

# 15. URL-Encoded Data

For URL-encoded form bodies:

``` js
app.use(express.urlencoded({
  extended: true
}));
```

Example:

``` text
name=Alice&age=25
```

------------------------------------------------------------------------

# 16. Request Object

## 16.1 Important properties of `req`

Common properties:

``` js
req.params
req.query
req.body
req.headers
req.cookies
req.method
req.originalUrl
req.path
req.protocol
req.ip
```

Example:

``` js
app.get("/users/:id", (req, res) => {
  console.log(req.params);
  console.log(req.query);
  console.log(req.headers);
});
```

------------------------------------------------------------------------

# 17. Response Object

## 17.1 Important `res` methods

``` js
res.send()
res.json()
res.status()
res.end()
res.redirect()
res.sendFile()
res.set()
res.cookie()
res.clearCookie()
```

Example:

``` js
res.status(201).json({
  id: 123,
  message: "Created"
});
```

------------------------------------------------------------------------

# 18. `res.send()` vs `res.json()`

`res.send()` can send strings, buffers, objects, etc., with Express
determining an appropriate response behavior.

``` js
res.send("Hello");
```

`res.json()` explicitly sends JSON:

``` js
res.json({
  message: "Hello"
});
```

For JSON APIs, `res.json()` makes intent clear.

------------------------------------------------------------------------

# 19. Status Codes

Common API status codes:

``` text
200 OK
201 Created
204 No Content

400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Content
429 Too Many Requests

500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

### Common trap

`401` generally means authentication is required/failed.

`403` means the server understood the request but refuses to authorize
it.

------------------------------------------------------------------------

# 20. REST API Design

## 20.1 Example REST API

``` text
GET    /users
GET    /users/:id
POST   /users
PATCH  /users/:id
DELETE /users/:id
```

Prefer resource-oriented URLs rather than action-heavy URLs.

Less ideal:

``` text
POST /createUser
POST /deleteUser
```

More REST-oriented:

``` text
POST /users
DELETE /users/:id
```

The exact API design should match the application's contract.

------------------------------------------------------------------------

# 21. Controllers

## 21.1 What is a controller?

A controller handles HTTP-level concerns:

-   Read request
-   Validate/normalize input
-   Call business logic
-   Construct response

Example:

``` js
async function getUser(req, res, next) {
  try {
    const user = await userService.getById(req.params.id);

    res.json(user);
  } catch (error) {
    next(error);
  }
}
```

Controllers should generally avoid containing large amounts of business
logic.

------------------------------------------------------------------------

# 22. Services

## 22.1 What belongs in a service?

Services contain business logic.

``` js
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  async getById(id) {
    return this.userRepository.findById(id);
  }
}
```

Architecture:

``` text
Route
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
Database
```

------------------------------------------------------------------------

# 23. Repository Layer

## 23.1 What is a repository?

A repository abstracts data-access operations.

``` js
class UserRepository {
  async findById(id) {
    return db.query(
      "SELECT * FROM users WHERE id = $1",
      [id]
    );
  }
}
```

Benefits:

-   Easier testing
-   Separation of concerns
-   Centralized data access
-   Easier database changes

Don't introduce layers mechanically; use them where they improve
maintainability.

------------------------------------------------------------------------

# 24. Error Handling

## 24.1 How does Express handle errors?

Synchronous route errors can be caught by Express.

``` js
app.get("/", (req, res) => {
  throw new Error("Something failed");
});
```

For asynchronous code, use proper Promise rejection handling. Modern
Express versions can forward rejected route-handler Promises to error
middleware.

A robust application can still use explicit `try/catch` when it needs
local recovery or transformation.

------------------------------------------------------------------------

# 25. Error-Handling Middleware

Error middleware has four parameters:

``` js
app.use((err, req, res, next) => {
  console.error(err);

  res.status(500).json({
    error: "Internal Server Error"
  });
});
```

Register it after routes.

------------------------------------------------------------------------

# 26. Centralized Error Handling

A scalable API should avoid repeating:

``` js
res.status(500).json(...)
```

everywhere.

Instead:

``` js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
  }
}
```

Throw:

``` js
throw new AppError("User not found", 404);
```

Central handler:

``` js
app.use((err, req, res, next) => {
  const status = err.statusCode || 500;

  res.status(status).json({
    error: err.message
  });
});
```

In production, don't expose stack traces or sensitive internal details
to clients.

------------------------------------------------------------------------

# 27. 404 Handling

A 404 handler should normally be placed after all routes:

``` js
app.use((req, res) => {
  res.status(404).json({
    error: "Route not found"
  });
});
```

Then the error handler can follow it:

``` js
app.use(errorHandler);
```

------------------------------------------------------------------------

# 28. Authentication Middleware

Example:

``` js
function authenticate(req, res, next) {
  const token = req.headers.authorization;

  if (!token) {
    return res.sendStatus(401);
  }

  // Verify token...
  req.user = decodedUser;

  next();
}
```

Use:

``` js
app.get(
  "/profile",
  authenticate,
  getProfile
);
```

------------------------------------------------------------------------

# 29. Authentication vs Authorization

Authentication:

> Who is the user?

Authorization:

> What is this user allowed to do?

Example:

``` js
function requireAdmin(req, res, next) {
  if (req.user.role !== "admin") {
    return res.sendStatus(403);
  }

  next();
}
```

Flow:

``` text
Request
 ↓
Authentication
 ↓
Authorization
 ↓
Controller
```

------------------------------------------------------------------------

# 30. JWT Middleware

Conceptual example:

``` js
const jwt = require("jsonwebtoken");

function authenticate(req, res, next) {
  const auth = req.headers.authorization;

  if (!auth?.startsWith("Bearer ")) {
    return res.sendStatus(401);
  }

  const token = auth.slice(7);

  try {
    req.user = jwt.verify(
      token,
      process.env.JWT_SECRET
    );

    next();
  } catch {
    res.sendStatus(401);
  }
}
```

Production considerations:

-   Short token lifetimes
-   Key rotation
-   Secure key storage
-   Issuer/audience validation where appropriate
-   Revocation strategy for high-risk use cases
-   Secure browser storage strategy

------------------------------------------------------------------------

# 31. Session-Based Authentication

Typical flow:

``` text
Login
 ↓
Server creates session
 ↓
Session ID in cookie
 ↓
Browser sends cookie
 ↓
Server retrieves session
```

For multiple Express instances, session storage generally needs shared
coordination such as Redis or a database rather than process-local
memory.

------------------------------------------------------------------------

# 32. JWT vs Sessions

  -----------------------------------------------------------------------
  Feature                 JWT                     Session
  ----------------------- ----------------------- -----------------------
  Server-side session     Not inherently required Yes
  state                                           

  Easy revocation         More complicated        Usually easier

  Horizontal scaling      Can be stateless        Requires shared session
                                                  store

  Token size              Can be larger           Cookie often contains
                                                  short ID

  Best choice             Depends on architecture Depends on architecture
  -----------------------------------------------------------------------

Neither is universally better.

------------------------------------------------------------------------

# 33. Cookies

Set a cookie:

``` js
res.cookie("sessionId", sessionId, {
  httpOnly: true,
  secure: true,
  sameSite: "lax"
});
```

Important attributes:

-   `HttpOnly`
-   `Secure`
-   `SameSite`
-   `Max-Age` / `Expires`
-   `Domain`
-   `Path`

------------------------------------------------------------------------

# 34. CORS

## 34.1 What is CORS?

CORS (Cross-Origin Resource Sharing) is a browser security mechanism
that controls whether JavaScript running on one origin can access
resources from another origin.

Example:

``` js
const cors = require("cors");

app.use(cors({
  origin: "https://example.com"
}));
```

Avoid blindly allowing all origins for credentialed/sensitive APIs.

------------------------------------------------------------------------

# 35. `app.use()` vs HTTP Method Routes

`app.use()` is commonly used for middleware or mounted routers:

``` js
app.use("/api", apiRouter);
```

Method-specific routes match a particular HTTP method:

``` js
app.get("/users", handler);
app.post("/users", handler);
```

------------------------------------------------------------------------

# 36. `app.use('/api', router)`

Suppose:

``` js
router.get("/users", handler);

app.use("/api", router);
```

The final route is:

``` text
GET /api/users
```

This is useful for versioning and grouping:

``` js
app.use("/api/v1/users", userRouter);
```

------------------------------------------------------------------------

# 37. API Versioning

Common strategies:

``` text
/api/v1/users
/api/v2/users
```

or versioning through headers/media types.

URL versioning is simple and explicit:

``` js
app.use("/api/v1", v1Router);
app.use("/api/v2", v2Router);
```

Version when you need incompatible API contracts; don't create versions
for every small change.

------------------------------------------------------------------------

# 38. Request Validation

Never trust:

``` js
req.body
req.query
req.params
```

Validate input.

Example:

``` js
function validateCreateUser(req, res, next) {
  const { email, age } = req.body;

  if (
    typeof email !== "string" ||
    !email.includes("@") ||
    !Number.isInteger(age)
  ) {
    return res.status(400).json({
      error: "Invalid input"
    });
  }

  next();
}
```

In production, schema validators are usually preferable for complex
schemas.

------------------------------------------------------------------------

# 39. Validation Middleware

A reusable pattern:

``` js
function validate(schema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);

    if (!result.success) {
      return res.status(400).json({
        error: "Validation failed"
      });
    }

    req.body = result.data;
    next();
  };
}
```

This separates validation from business logic.

------------------------------------------------------------------------

# 40. Sanitization

Validation answers:

> Is this input allowed?

Sanitization/transformation answers:

> How should this input be normalized safely?

Example:

``` js
const email = req.body.email
  .trim()
  .toLowerCase();
```

Do not blindly sanitize everything; transformations should match the
field's semantics.

------------------------------------------------------------------------

# 41. SQL Injection

Bad:

``` js
const query = `
  SELECT *
  FROM users
  WHERE id = ${req.params.id}
`;
```

Better:

``` js
const result = await db.query(
  "SELECT * FROM users WHERE id = $1",
  [req.params.id]
);
```

Use parameterized queries or safe query builders/ORMs.

------------------------------------------------------------------------

# 42. NoSQL Injection

Avoid blindly passing user-controlled objects into database filters:

``` js
db.users.findOne(req.body);
```

Instead explicitly construct the expected query:

``` js
db.users.findOne({
  email: req.body.email
});
```

Validate types and allowed fields.

------------------------------------------------------------------------

# 43. XSS

Cross-Site Scripting occurs when untrusted content becomes executable
browser content.

Risky:

``` js
element.innerHTML = userInput;
```

For plain text:

``` js
element.textContent = userInput;
```

For server APIs, also avoid reflecting untrusted HTML into HTML
responses.

------------------------------------------------------------------------

# 44. CSRF

Cross-Site Request Forgery causes a victim's browser to send an unwanted
authenticated request.

Important defenses:

-   SameSite cookies
-   CSRF tokens
-   Origin validation where appropriate
-   Avoiding unsafe cross-site credential flows

CSRF is especially relevant when authentication relies on cookies
automatically sent by the browser.

------------------------------------------------------------------------

# 45. Helmet / Security Headers

Security headers reduce several classes of browser-side attacks.

Example:

``` js
const helmet = require("helmet");

app.use(helmet());
```

Review the resulting policy for your application's requirements rather
than assuming defaults are perfect for every application.

------------------------------------------------------------------------

# 46. Rate Limiting

Rate limiting protects endpoints from excessive traffic.

Example with a rate-limiting middleware:

``` js
const rateLimit = require("express-rate-limit");

app.use("/api", rateLimit({
  windowMs: 60 * 1000,
  limit: 100
}));
```

For horizontally scaled systems, in-memory counters are not sufficient
for globally consistent limits; use shared storage or an API
gateway/distributed limiter.

------------------------------------------------------------------------

# 47. Rate Limiting Algorithms

Know these for interviews:

### Fixed window

``` text
00:00 ───────── 00:59
limit = 100
```

Simple but can allow boundary bursts.

### Sliding window

Tracks usage over a moving time window.

### Token bucket

``` text
Bucket
 ↓
Tokens refill
 ↓
Request consumes token
```

Allows controlled bursts while limiting average rate.

### Leaky bucket

Processes requests at a controlled rate.

------------------------------------------------------------------------

# 48. Logging Middleware

Simple:

``` js
app.use((req, res, next) => {
  const start = Date.now();

  res.on("finish", () => {
    console.log({
      method: req.method,
      path: req.originalUrl,
      status: res.statusCode,
      durationMs: Date.now() - start
    });
  });

  next();
});
```

Production logging should generally be structured rather than relying
only on strings.

------------------------------------------------------------------------

# 49. Request IDs

Generate/request-propagate a request ID:

``` js
const crypto = require("node:crypto");

app.use((req, res, next) => {
  const requestId =
    req.get("x-request-id") ||
    crypto.randomUUID();

  req.requestId = requestId;
  res.set("x-request-id", requestId);

  next();
});
```

Useful for tracing a request through logs and downstream services.

------------------------------------------------------------------------

# 50. Health Checks

## Liveness

``` js
app.get("/health/live", (req, res) => {
  res.sendStatus(200);
});
```

## Readiness

``` js
app.get("/health/ready", async (req, res) => {
  const ready = await checkDependencies();

  if (!ready) {
    return res.sendStatus(503);
  }

  res.sendStatus(200);
});
```

Liveness asks whether the process is alive.

Readiness asks whether it should receive traffic.

------------------------------------------------------------------------

# 51. Graceful Shutdown

A production Express application should handle termination signals.

``` js
const server = app.listen(3000);

process.on("SIGTERM", () => {
  server.close(() => {
    console.log("HTTP server closed");
    process.exit(0);
  });
});
```

A stronger implementation also closes:

-   Database pools
-   Redis connections
-   Queue consumers
-   Other resources

and uses a shutdown deadline.

------------------------------------------------------------------------

# 52. Why Is Graceful Shutdown Important?

Without it:

``` text
Deployment
 ↓
Process killed
 ↓
Active requests fail
 ↓
Partial operations
```

With graceful shutdown:

``` text
SIGTERM
 ↓
Stop accepting new requests
 ↓
Finish in-flight work
 ↓
Close resources
 ↓
Exit
```

This is especially important in containers and rolling deployments.

------------------------------------------------------------------------

# 53. Async Error Handling

A clean Express route:

``` js
app.get("/users/:id", async (req, res, next) => {
  try {
    const user = await userService.getUser(req.params.id);

    res.json(user);
  } catch (error) {
    next(error);
  }
});
```

For applications using modern Express async error propagation, rejected
async handlers can be forwarded to error middleware. Explicit
`try/catch` is still useful when you need local handling or want to add
context.

------------------------------------------------------------------------

# 54. Error Classes

``` js
class NotFoundError extends Error {
  constructor(message = "Not found") {
    super(message);
    this.statusCode = 404;
    this.code = "NOT_FOUND";
  }
}
```

Then:

``` js
throw new NotFoundError("User not found");
```

Central handler:

``` js
app.use((err, req, res, next) => {
  const status = err.statusCode || 500;

  res.status(status).json({
    error: {
      code: err.code || "INTERNAL_ERROR",
      message: status >= 500
        ? "Internal Server Error"
        : err.message
    }
  });
});
```

------------------------------------------------------------------------

# 55. Avoiding Double Responses

Bad:

``` js
app.get("/", (req, res) => {
  if (!authorized) {
    res.sendStatus(403);
  }

  res.json({ data: "secret" });
});
```

The second response can cause:

``` text
Error: Cannot set headers after they are sent
```

Better:

``` js
if (!authorized) {
  return res.sendStatus(403);
}

res.json({ data: "secret" });
```

------------------------------------------------------------------------

# 56. `res.headersSent`

You can check whether headers have already been sent:

``` js
if (res.headersSent) {
  return next(err);
}
```

This can be useful in centralized error handling.

------------------------------------------------------------------------

# 57. Static Files

Express can serve static assets:

``` js
app.use(
  express.static("public")
);
```

A file:

``` text
public/index.html
```

may then be served as a static resource.

For large-scale production systems, static assets are often better
served through a CDN/object-storage layer.

------------------------------------------------------------------------

# 58. Compression

HTTP compression can reduce response size.

``` js
const compression = require("compression");

app.use(compression());
```

Trade-offs:

-   CPU cost
-   Reduced bandwidth
-   Better latency for compressible payloads

Avoid compressing data that is already compressed.

------------------------------------------------------------------------

# 59. Body Size Limits

Don't accept unlimited request bodies.

``` js
app.use(express.json({
  limit: "1mb"
}));
```

This helps reduce memory exhaustion and abuse.

------------------------------------------------------------------------

# 60. File Uploads

For multipart uploads, use a suitable multipart parser and enforce
limits.

Important controls:

-   Maximum file size
-   Allowed types
-   Authentication
-   Storage limits
-   Filename/path safety
-   Malware scanning where required

Prefer streaming large files to object storage rather than buffering
entire files in Node memory.

------------------------------------------------------------------------

# 61. Pagination

Offset:

``` text
GET /users?limit=20&offset=100
```

Cursor:

``` text
GET /users?limit=20&cursor=abc
```

Cursor pagination is often more stable and efficient for large, changing
datasets.

------------------------------------------------------------------------

# 62. Filtering and Sorting

Example:

``` text
GET /users?
  role=admin
  &sort=createdAt
  &order=desc
```

Never directly concatenate untrusted sort fields into SQL.

Use an allowlist:

``` js
const allowedSortFields = {
  created: "created_at",
  name: "name"
};

const sortColumn =
  allowedSortFields[req.query.sort] || "created_at";
```

------------------------------------------------------------------------

# 63. API Response Design

A consistent response format can make clients easier to maintain.

Example:

``` json
{
  "data": {
    "id": 123,
    "name": "Alice"
  }
}
```

Error:

``` json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User not found"
  }
}
```

Consistency matters more than any one universal format.

------------------------------------------------------------------------

# 64. Idempotency

For operations with side effects, clients may retry because of network
failures.

Example:

``` text
POST /payments
Idempotency-Key: abc123
```

Server stores:

``` text
abc123 → previous result
```

If the request is repeated, return the stored result rather than
performing the side effect again.

This is especially important for:

-   Payments
-   Orders
-   Resource creation
-   External API calls

------------------------------------------------------------------------

# 65. Database Connection Pooling

Don't create a new DB connection per request.

Bad:

``` text
Request
 ↓
Open DB connection
 ↓
Query
 ↓
Close
```

Better:

``` text
Node process
     ↓
Connection pool
 |   |   |   |
 DB  DB  DB  DB
```

Example:

``` js
const { Pool } = require("pg");

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20
});
```

Pool size must be tuned against database capacity and total number of
application instances.

------------------------------------------------------------------------

# 66. Transactions in Express

The Express layer should generally delegate transaction logic to the
service/data-access layer.

Example:

``` js
async function transfer(fromId, toId, amount) {
  const client = await pool.connect();

  try {
    await client.query("BEGIN");

    await client.query(
      "UPDATE accounts SET balance = balance - $1 WHERE id = $2",
      [amount, fromId]
    );

    await client.query(
      "UPDATE accounts SET balance = balance + $1 WHERE id = $2",
      [amount, toId]
    );

    await client.query("COMMIT");
  } catch (error) {
    await client.query("ROLLBACK");
    throw error;
  } finally {
    client.release();
  }
}
```

------------------------------------------------------------------------

# 67. Redis in Express

Common uses:

-   Caching
-   Rate limiting
-   Sessions
-   Distributed coordination
-   Queues
-   Counters

Cache-aside example:

``` js
async function getUser(id) {
  const key = `user:${id}`;

  const cached = await redis.get(key);

  if (cached) {
    return JSON.parse(cached);
  }

  const user = await db.findUser(id);

  await redis.set(
    key,
    JSON.stringify(user),
    { EX: 60 }
  );

  return user;
}
```

------------------------------------------------------------------------

# 68. Cache Stampede

Suppose:

``` text
Cache entry expires
       ↓
1000 requests arrive
       ↓
1000 DB queries
```

Solutions:

-   Request coalescing
-   Locks
-   Early refresh
-   Randomized TTL
-   Stale-while-revalidate

------------------------------------------------------------------------

# 69. Request Deduplication

Share an in-flight Promise:

``` js
const inFlight = new Map();

function getUser(id) {
  if (inFlight.has(id)) {
    return inFlight.get(id);
  }

  const promise = fetchUser(id)
    .finally(() => {
      inFlight.delete(id);
    });

  inFlight.set(id, promise);

  return promise;
}
```

This prevents duplicate concurrent work within one process.

For multiple instances, coordinate through a shared mechanism if global
deduplication is required.

------------------------------------------------------------------------

# 70. Timeouts

External services should not be allowed to hang indefinitely.

With `fetch`:

``` js
const controller = new AbortController();

const timeout = setTimeout(() => {
  controller.abort();
}, 3000);

try {
  const response = await fetch(url, {
    signal: controller.signal
  });

  return await response.json();
} finally {
  clearTimeout(timeout);
}
```

Timeouts should be designed around an overall request deadline.

------------------------------------------------------------------------

# 71. Retries

Don't retry every failure.

Retry carefully for:

-   Temporary network failures
-   Rate limits when instructed
-   Certain 5xx failures
-   Transient infrastructure errors

Avoid retries for:

-   Validation errors
-   Authentication failures
-   Authorization failures
-   Permanent business errors

Use:

``` text
Retry
 ↓
Exponential backoff
 ↓
Jitter
 ↓
Maximum attempts/deadline
```

------------------------------------------------------------------------

# 72. Retry Storms

Bad:

``` text
Dependency fails
 ↓
1000 requests fail
 ↓
1000 immediate retries
 ↓
Dependency gets even more traffic
 ↓
Fails harder
```

Use exponential backoff and jitter.

------------------------------------------------------------------------

# 73. Circuit Breaker

A circuit breaker protects your service from repeatedly calling an
unhealthy dependency.

``` text
CLOSED
  |
  | failures exceed threshold
  ↓
OPEN
  |
  | wait
  ↓
HALF-OPEN
  |
  | success → CLOSED
  | failure → OPEN
```

When open, fail fast or use a fallback.

------------------------------------------------------------------------

# 74. Bulkheads

Isolate resource pools for different dependencies.

``` text
Express service
 |
 +-- Payment concurrency = 20
 |
 +-- Email concurrency = 10
 |
 +-- Search concurrency = 30
```

If email becomes slow, it doesn't consume every resource needed by
payments.

------------------------------------------------------------------------

# 75. Queues and Background Jobs

Don't perform long-running work inside the HTTP request if it doesn't
need to block the client.

Bad:

``` text
POST /report
 ↓
Generate huge report
 ↓
Upload
 ↓
Send email
 ↓
Response
```

Better:

``` text
POST /report
 ↓
Create job
 ↓
Return 202
 ↓
Worker processes job
```

Response:

``` js
res.status(202).json({
  jobId
});
```

------------------------------------------------------------------------

# 76. Why Return `202 Accepted`?

`202 Accepted` is useful when the request has been accepted for
processing but work is not complete.

Example:

``` text
POST /reports
→ 202 Accepted
{
  "jobId": "123"
}
```

Client can later check:

``` text
GET /reports/jobs/123
```

------------------------------------------------------------------------

# 77. WebSockets with Express

Express itself handles HTTP request/response routing; WebSocket
connections are typically integrated with a WebSocket library/runtime.

Use WebSockets for:

-   Chat
-   Real-time collaboration
-   Live dashboards
-   Multiplayer
-   Bidirectional updates

Architecture:

``` text
Browser
   ↕
WebSocket server
   ↕
Redis/pub-sub or message system
   ↕
Other instances
```

------------------------------------------------------------------------

# 78. SSE with Express

Server-Sent Events provide a long-lived HTTP connection from which the
server can push events to the client.

Example:

``` js
app.get("/events", (req, res) => {
  res.set({
    "Content-Type": "text/event-stream",
    "Cache-Control": "no-cache",
    "Connection": "keep-alive"
  });

  res.flushHeaders();

  const timer = setInterval(() => {
    res.write(`data: ${JSON.stringify({
      time: Date.now()
    })}\n\n`);
  }, 1000);

  req.on("close", () => {
    clearInterval(timer);
  });
});
```

Remember to clean up resources when clients disconnect.

------------------------------------------------------------------------

# 79. Express Behind a Reverse Proxy

Production architecture often looks like:

``` text
Internet
   ↓
CDN / Load Balancer
   ↓
Reverse Proxy
   ↓
Express instances
   ↓
Redis / DB / Queue
```

The proxy may handle:

-   TLS termination
-   Load balancing
-   Compression/CDN integration
-   Rate limiting
-   Routing

------------------------------------------------------------------------

# 80. `trust proxy`

When Express runs behind a proxy, configure proxy trust carefully when
you need accurate client IP/protocol information.

``` js
app.set("trust proxy", 1);
```

The correct configuration depends on your network topology.

This affects values such as:

``` js
req.ip
req.protocol
```

and secure-cookie behavior.

Don't blindly enable proxy trust from arbitrary networks.

------------------------------------------------------------------------

# 81. Horizontal Scaling

Express applications should generally be designed to be stateless.

``` text
             Load Balancer
          /       |       \
      Express   Express   Express
         |         |         |
         +---------+---------+
                   |
             Shared services
          DB / Redis / Queue
```

Avoid relying on process-local state for data that must be shared
between instances.

------------------------------------------------------------------------

# 82. Why Is In-Memory Session Storage Bad for Multiple Instances?

Suppose:

``` text
User
 ↓
Server A
 ↓
session stored in A's memory
```

Next request:

``` text
User
 ↓
Server B
 ↓
session missing
```

Use shared session storage or a different stateless authentication
architecture.

------------------------------------------------------------------------

# 83. Sticky Sessions

Sticky sessions route a user to the same server.

This can reduce session-sharing problems but introduces trade-offs:

-   Uneven load
-   More complicated failover
-   Reduced flexibility

Prefer shared state or stateless architecture where practical instead of
relying on stickiness as the primary solution.

------------------------------------------------------------------------

# 84. Compression vs CDN

For static assets:

``` text
Client
 ↓
CDN
 ↓
Object storage
```

is often preferable to making Express serve every static asset.

Express should generally focus on dynamic application logic.

------------------------------------------------------------------------

# 85. API Security Checklist

A production Express API should consider:

``` text
HTTPS
Authentication
Authorization
Input validation
Rate limiting
Secure cookies
CORS policy
Security headers
Body-size limits
Parameterized queries
Secret management
Dependency security
Safe error responses
Request logging
Audit logging for sensitive operations
```

------------------------------------------------------------------------

# 86. Secret Management

Bad:

``` js
const JWT_SECRET = "my-super-secret";
```

Better:

``` js
const JWT_SECRET = process.env.JWT_SECRET;
```

For production, use a proper secret-management system when available.

Never commit credentials to source control.

------------------------------------------------------------------------

# 87. Environment Configuration

Example:

``` js
const config = {
  port: Number(process.env.PORT || 3000),
  databaseUrl: process.env.DATABASE_URL,
  redisUrl: process.env.REDIS_URL
};

if (!config.databaseUrl) {
  throw new Error("DATABASE_URL is required");
}
```

Fail fast on missing critical configuration.

------------------------------------------------------------------------

# 88. Testing Express APIs

## Unit test

Test a service independently:

``` js
expect(add(2, 3)).toBe(5);
```

## Integration/API test

Test:

``` text
HTTP request
 ↓
Express
 ↓
middleware
 ↓
service
 ↓
database/test DB
```

Example using a request-testing library:

``` js
const response = await request(app)
  .get("/users/1");

expect(response.status).toBe(200);
```

Test behavior rather than implementation details.

------------------------------------------------------------------------

# 89. Mocking Dependencies

Controller:

``` js
function createUserController(userService) {
  return async (req, res, next) => {
    try {
      const user = await userService.create(req.body);
      res.status(201).json(user);
    } catch (error) {
      next(error);
    }
  };
}
```

Test with fake service:

``` js
const fakeService = {
  create: async () => ({
    id: 1,
    name: "Alice"
  })
};
```

Dependency injection makes isolated testing easier.

------------------------------------------------------------------------

# 90. Dependency Injection

Instead of:

``` js
class UserService {
  constructor() {
    this.db = new Database();
  }
}
```

prefer:

``` js
class UserService {
  constructor(db) {
    this.db = db;
  }
}
```

Then testing can use:

``` js
const fakeDb = {
  findUser: async () => ({ id: 1 })
};

const service = new UserService(fakeDb);
```

------------------------------------------------------------------------

# 91. API Documentation

Document:

-   Endpoints
-   Parameters
-   Request body
-   Authentication
-   Response schema
-   Error responses
-   Status codes
-   Pagination
-   Rate limits

OpenAPI is a common standard for describing HTTP APIs.

------------------------------------------------------------------------

# 92. API Idempotency and HTTP Methods

Typical HTTP semantics:

``` text
GET    → safe/read
PUT    → generally idempotent
DELETE → generally idempotent
POST   → not inherently idempotent
PATCH  → depends on operation
```

Actual behavior depends on implementation.

------------------------------------------------------------------------

# 93. ETags and Conditional Requests

HTTP caching can use ETags:

``` js
res.set("ETag", `"version-123"`);
```

Clients may send:

``` text
If-None-Match
```

If content hasn't changed:

``` text
304 Not Modified
```

This can reduce bandwidth and response work.

------------------------------------------------------------------------

# 94. Cache-Control

Example:

``` js
res.set(
  "Cache-Control",
  "public, max-age=60"
);
```

For sensitive dynamic responses, caching policy should be carefully
controlled.

------------------------------------------------------------------------

# 95. Request Timeout Middleware

Conceptual middleware:

``` js
function timeout(ms) {
  return (req, res, next) => {
    const timer = setTimeout(() => {
      if (!res.headersSent) {
        res.status(408).json({
          error: "Request timeout"
        });
      }
    }, ms);

    res.on("finish", () => {
      clearTimeout(timer);
    });

    next();
  };
}
```

For downstream calls, prefer cancellation/deadline propagation rather
than merely sending a timeout response while underlying work continues.

------------------------------------------------------------------------

# 96. Concurrency Limiting

If an endpoint triggers expensive operations:

``` js
await Promise.all(
  items.map(processItem)
);
```

could create thousands of concurrent operations.

Better:

``` text
10,000 jobs
   ↓
Concurrency = 20
   ↓
20 active
   ↓
Completed job → next
```

This protects databases and downstream APIs.

------------------------------------------------------------------------

# 97. Avoiding N+1 Queries

Bad:

``` text
GET /users
 ↓
Query users
 ↓
For each user:
  query orders
```

For 100 users:

``` text
101 queries
```

Better:

``` text
Query users
 ↓
Batch order query
 ↓
Group orders by user
```

Or use appropriate joins/batching/data-loader patterns.

------------------------------------------------------------------------

# 98. Large Response Handling

Avoid building huge response objects unnecessarily.

Bad:

``` js
const millions = await db.getAllUsers();

res.json(millions);
```

Better:

-   Pagination
-   Streaming where appropriate
-   Filtering
-   Field selection
-   Asynchronous export jobs

------------------------------------------------------------------------

# 99. Large Request Handling

Protect your API:

``` js
app.use(express.json({
  limit: "1mb"
}));
```

For huge uploads, use streaming/multipart processing rather than
buffering everything in memory.

------------------------------------------------------------------------

# 100. Memory Leaks in Express

Common causes:

-   Global caches
-   Unbounded Maps
-   Timers
-   Event listeners
-   Long-lived closures
-   Request data stored globally
-   In-memory sessions
-   Large objects retained by references

Bad:

``` js
const requests = [];

app.use((req, res, next) => {
  requests.push(req);
  next();
});
```

This can retain request objects indefinitely.

------------------------------------------------------------------------

# 101. Observability

Production Express services should expose:

### Logs

``` text
requestId
method
route
status
duration
error
```

### Metrics

``` text
requests/sec
error rate
p50 latency
p95 latency
p99 latency
CPU
memory
event-loop delay
DB pool utilization
```

### Traces

Track a request across services.

``` text
API
 ↓
User service
 ↓
Payment service
 ↓
Database
```

------------------------------------------------------------------------

# 102. Structured Logging

Instead of:

``` js
console.log("User failed");
```

prefer structured events:

``` js
logger.error({
  requestId: req.requestId,
  userId,
  error: error.message
});
```

Never log passwords, access tokens, secret keys, or sensitive
information unnecessarily.

------------------------------------------------------------------------

# 103. API Latency Debugging

If an Express endpoint becomes slow:

``` text
Measure
 ↓
Endpoint
 ↓
p95/p99
 ↓
DB latency
 ↓
External API latency
 ↓
Event-loop delay
 ↓
CPU
 ↓
Memory/GC
 ↓
Connection pools
 ↓
Recent deployment
```

Fix the measured bottleneck.

------------------------------------------------------------------------

# 104. CPU-Heavy Work

Bad:

``` js
app.get("/report", (req, res) => {
  const result = expensiveCPUCalculation();
  res.json(result);
});
```

If the calculation is CPU-heavy, it can block the event loop.

Better options:

-   Worker Threads
-   Background jobs
-   Separate services
-   Child processes
-   Precomputation

------------------------------------------------------------------------

# 105. Worker Threads with Express

Concept:

``` text
HTTP request
   ↓
Express
   ↓
Worker thread
   ↓
CPU-intensive calculation
   ↓
Result
   ↓
HTTP response
```

Use workers when CPU-intensive JavaScript needs to execute without
blocking the main event loop.

------------------------------------------------------------------------

# 106. Background Jobs

Example architecture:

``` text
Client
  ↓
Express API
  ↓
Queue
  ↓
Worker
  ↓
Database / object storage / external service
```

The API responds quickly while workers handle expensive tasks.

------------------------------------------------------------------------

# 107. Load Shedding

When the system is overloaded, it may be better to reject lower-priority
work than let everything become slow.

Example:

``` text
High traffic
   ↓
Capacity reached
   ↓
Reject optional requests
   ↓
Protect critical endpoints
```

Possible response:

``` text
429 Too Many Requests
```

or:

``` text
503 Service Unavailable
```

depending on the situation.

------------------------------------------------------------------------

# 108. Graceful Degradation

If recommendations fail:

``` text
Product request
 ↓
Product data succeeds
 ↓
Recommendation service fails
 ↓
Return product without recommendations
```

This preserves core functionality.

------------------------------------------------------------------------

# 109. Reverse Proxy and Load Balancer

A production Express deployment might be:

``` text
             Internet
                ↓
             CDN/WAF
                ↓
         Load Balancer
          /     |     \
       API     API     API
        |       |       |
        +-------+-------+
                |
        DB / Redis / Queue
```

The Express processes should ideally be stateless.

------------------------------------------------------------------------

# 110. SDE2: Design a Production Express API

Suppose asked:

> "Design a scalable user API."

A strong answer:

``` text
                 Client
                   |
                CDN/WAF
                   |
              Load Balancer
                   |
        +----------+----------+
        |          |          |
      API-1      API-2      API-3
        |          |          |
        +----------+----------+
                   |
          +--------+--------+
          |                 |
        Redis             PostgreSQL
          |
       Cache
                   |
                 Queue
                   |
                Workers
```

Discuss:

-   Authentication
-   Authorization
-   Validation
-   Rate limiting
-   Caching
-   DB indexes
-   Connection pools
-   Pagination
-   Timeouts
-   Retries
-   Idempotency
-   Observability
-   Graceful shutdown
-   Horizontal scaling

------------------------------------------------------------------------

# 111. SDE2: Database Overload

If the DB becomes overloaded:

First investigate:

``` text
Slow queries?
N+1?
Missing indexes?
Too many connections?
Traffic spike?
Retry storm?
```

Possible solutions:

``` text
Optimize queries
 ↓
Indexes
 ↓
Connection pool tuning
 ↓
Caching
 ↓
Rate limiting
 ↓
Queue background work
 ↓
Read replicas where appropriate
```

Don't simply increase the DB connection pool; that can make overload
worse.

------------------------------------------------------------------------

# 112. SDE2: External Service Failure

Suppose your Express API calls a payment service.

Robust flow:

``` text
Request
 ↓
Validate
 ↓
Authentication
 ↓
Timeout
 ↓
Payment service
 ↓
Success?
 /    \
yes    no
 |      |
return  retryable?
         |
       backoff
         |
       retry
         |
     circuit breaker
         |
       fallback/fail
```

For side effects, add idempotency.

------------------------------------------------------------------------

# 113. SDE2: Traffic Spike

If traffic increases 10x:

``` text
Traffic spike
 ↓
Load balancer
 ↓
Horizontal scaling
 ↓
Cache
 ↓
Rate limiting
 ↓
Concurrency limits
 ↓
Queue
 ↓
Protect DB/downstreams
```

Discuss the bottleneck rather than assuming application servers are the
only scaling problem.

------------------------------------------------------------------------

# 114. SDE2: Memory Leak Investigation

Process:

``` text
Memory increases
 ↓
Check RSS/heap
 ↓
Check GC behavior
 ↓
Heap snapshots
 ↓
Compare snapshots
 ↓
Retaining paths
 ↓
Find cache/listener/timer/reference
 ↓
Fix
 ↓
Load test
 ↓
Verify stable memory
```

------------------------------------------------------------------------

# 115. SDE2: Event Loop Blocking

If p99 latency suddenly rises:

``` text
p99 latency ↑
      |
event-loop delay ↑
      |
CPU ↑
      |
synchronous/CPU-heavy code?
      |
worker threads/background jobs
```

Potential culprits:

-   Large JSON processing
-   Synchronous filesystem operations
-   CPU-heavy loops
-   Compression at excessive CPU cost
-   Expensive serialization
-   Bad regular expressions

------------------------------------------------------------------------

# 116. SDE2: Regex DoS

Certain pathological regular expressions can consume excessive CPU on
crafted input.

Bad conceptual example:

``` js
const regex = /(a+)+$/;
```

Mitigations:

-   Avoid unsafe regex patterns
-   Bound input sizes
-   Prefer safer parsers
-   Use timeouts/worker isolation where appropriate
-   Test regexes against adversarial input

------------------------------------------------------------------------

# 117. SDE2: Graceful Deployment

During deployment:

``` text
New version starts
       ↓
Readiness check passes
       ↓
Load balancer sends traffic
       ↓
Old instance receives SIGTERM
       ↓
Stops accepting new work
       ↓
Finishes active requests
       ↓
Closes resources
       ↓
Exits
```

This supports rolling deployments with minimal disruption.

------------------------------------------------------------------------

# 118. SDE2: Zero-Downtime API Evolution

Suppose database schema needs a breaking change.

Don't deploy:

``` text
Code expects new column
       ↓
DB old schema
       ↓
Failure
```

Use expand/contract:

``` text
1. Expand schema
2. Deploy backward-compatible code
3. Migrate/backfill
4. Switch reads/writes
5. Remove old schema later
```

This is a common production deployment pattern.

------------------------------------------------------------------------

# 119. SDE2: Idempotent Background Jobs

A job may run twice due to retries:

``` text
Job
 ↓
Worker processes
 ↓
Worker crashes before ack
 ↓
Job retried
```

Design the operation to be idempotent:

``` js
await db.query(`
  INSERT INTO processed_jobs(job_id)
  VALUES ($1)
  ON CONFLICT (job_id) DO NOTHING
`, [jobId]);
```

Then only process the side effect if the job wasn't already processed.

------------------------------------------------------------------------

# 120. SDE2: Distributed State

Avoid:

``` js
const activeUsers = new Map();
```

for state that must be consistent across multiple Express instances.

Use shared systems:

``` text
Express instances
       |
   Redis / DB
```

Process-local state is fine for local caches or ephemeral state when
consistency across instances is not required.

------------------------------------------------------------------------

# 121. Common Express Interview Coding Questions

## Q1. Write authentication middleware

``` js
function auth(req, res, next) {
  const token = req.headers.authorization;

  if (!token) {
    return res.sendStatus(401);
  }

  try {
    req.user = verifyToken(token);
    next();
  } catch {
    res.sendStatus(401);
  }
}
```

Follow-ups:

-   Where should the token come from?
-   How do you revoke tokens?
-   JWT vs session?
-   How do you handle refresh tokens?
-   How do you protect cookies?

------------------------------------------------------------------------

## Q2. Write role-based authorization middleware

``` js
function requireRole(role) {
  return (req, res, next) => {
    if (req.user?.role !== role) {
      return res.sendStatus(403);
    }

    next();
  };
}
```

Usage:

``` js
app.delete(
  "/users/:id",
  auth,
  requireRole("admin"),
  deleteUser
);
```

------------------------------------------------------------------------

## Q3. Write centralized error middleware

``` js
function errorHandler(err, req, res, next) {
  console.error(err);

  const status = err.statusCode || 500;

  res.status(status).json({
    error: {
      code: err.code || "INTERNAL_ERROR",
      message: status >= 500
        ? "Internal Server Error"
        : err.message
    }
  });
}
```

------------------------------------------------------------------------

## Q4. Write request logging middleware

``` js
function logger(req, res, next) {
  const start = Date.now();

  res.on("finish", () => {
    console.log({
      method: req.method,
      path: req.originalUrl,
      status: res.statusCode,
      duration: Date.now() - start
    });
  });

  next();
}
```

------------------------------------------------------------------------

## Q5. Write rate-limiting middleware conceptually

``` js
const requests = new Map();

function rateLimit(limit, windowMs) {
  return (req, res, next) => {
    const key = req.ip;
    const now = Date.now();

    const entry = requests.get(key) || {
      count: 0,
      start: now
    };

    if (now - entry.start >= windowMs) {
      entry.count = 0;
      entry.start = now;
    }

    entry.count++;
    requests.set(key, entry);

    if (entry.count > limit) {
      return res.sendStatus(429);
    }

    next();
  };
}
```

### SDE2 follow-up

Why is this insufficient in production?

Because each Express instance has its own counter.

For distributed rate limiting, use shared storage or an
edge/gateway/distributed rate limiter.

------------------------------------------------------------------------

# 122. Common Express Interview Traps

## Trap 1: Middleware order

``` js
app.get("/users", handler);

app.use(auth);
```

`auth` won't protect the route above it.

Register middleware appropriately before protected routes.

------------------------------------------------------------------------

## Trap 2: Forgetting `next()`

``` js
app.use((req, res, next) => {
  console.log("hello");
});
```

The request can hang.

------------------------------------------------------------------------

## Trap 3: Sending two responses

``` js
if (!user) {
  res.sendStatus(404);
}

res.json(user);
```

Use:

``` js
if (!user) {
  return res.sendStatus(404);
}
```

------------------------------------------------------------------------

## Trap 4: Trusting client input

Never assume:

``` js
req.body.role === "admin"
```

is safe.

Authorization must be enforced server-side.

------------------------------------------------------------------------

## Trap 5: In-memory state in a cluster

``` js
const sessions = new Map();
```

works only within one process.

It does not automatically synchronize across instances.

------------------------------------------------------------------------

# 123. SDE1 Express Questions

Be able to answer these quickly:

1.  What is Express?
2.  Express vs Node.js?
3.  What is middleware?
4.  Why does middleware order matter?
5.  What is `next()`?
6.  What is `express.Router()`?
7.  `app.use()` vs `app.get()`?
8.  Route params vs query params?
9.  How do you read JSON body?
10. `req.params` vs `req.query` vs `req.body`?
11. `res.send()` vs `res.json()`?
12. HTTP status codes?
13. How do you handle errors?
14. What is error-handling middleware?
15. How do you implement authentication?
16. Authentication vs authorization?
17. JWT vs session?
18. What is CORS?
19. What is rate limiting?
20. How do you validate input?
21. How do you prevent SQL injection?
22. What is Helmet?
23. What are cookies?
24. What is `express.static()`?
25. How do you test an Express API?
26. How do you use environment variables?
27. How do you handle file uploads?
28. How do you paginate APIs?
29. How do you structure an Express application?
30. How do you gracefully shut down an Express server?

------------------------------------------------------------------------

# 124. SDE2 Express Questions

Be able to reason about:

1.  How does Express middleware composition work internally?
2.  How would you design a large Express application?
3.  How do you handle async errors consistently?
4.  How do you prevent event-loop blocking?
5.  How do you scale Express horizontally?
6.  How do you handle sessions across instances?
7.  How do you design distributed rate limiting?
8.  How do you prevent cache stampedes?
9.  How do you implement request deduplication?
10. How do you implement idempotency?
11. How do you handle retries safely?
12. How do you propagate deadlines/timeouts?
13. How do you protect downstream services?
14. When would you use a queue?
15. How do you handle CPU-heavy work?
16. How do you diagnose memory leaks?
17. How do you monitor event-loop latency?
18. How do you implement graceful shutdown?
19. How do you design zero-downtime deployments?
20. How do you handle database schema changes?
21. How do you protect an API during traffic spikes?
22. How do you implement circuit breakers?
23. What is bulkheading?
24. How do you implement graceful degradation?
25. How do you design observability?
26. How do you prevent N+1 queries?
27. How do you handle large uploads/downloads?
28. How do you design real-time APIs?
29. How do you handle distributed state?
30. How do you design a reliable Express service?

------------------------------------------------------------------------

# 125. Express Production Checklist

## API

-   [ ] REST/resource design
-   [ ] Correct status codes
-   [ ] Consistent error format
-   [ ] Validation
-   [ ] Pagination
-   [ ] Filtering/sorting allowlists
-   [ ] API versioning where necessary

## Middleware

-   [ ] Logging
-   [ ] Authentication
-   [ ] Authorization
-   [ ] Validation
-   [ ] Rate limiting
-   [ ] CORS
-   [ ] Error handling
-   [ ] Request IDs

## Security

-   [ ] HTTPS
-   [ ] Secure cookies
-   [ ] Helmet/security headers
-   [ ] Input validation
-   [ ] SQL injection protection
-   [ ] NoSQL injection protection
-   [ ] XSS considerations
-   [ ] CSRF protection where relevant
-   [ ] Body size limits
-   [ ] Secret management
-   [ ] Dependency security

## Reliability

-   [ ] Timeouts
-   [ ] Retry policy
-   [ ] Exponential backoff
-   [ ] Jitter
-   [ ] Idempotency
-   [ ] Circuit breakers
-   [ ] Graceful degradation
-   [ ] Graceful shutdown
-   [ ] Health checks

## Performance

-   [ ] DB connection pooling
-   [ ] Caching
-   [ ] Request deduplication
-   [ ] Concurrency limits
-   [ ] Streams for large data
-   [ ] Avoid event-loop blocking
-   [ ] Compression where appropriate
-   [ ] CDN for static assets

## Observability

-   [ ] Structured logs
-   [ ] Metrics
-   [ ] Distributed tracing
-   [ ] Request IDs
-   [ ] Error tracking
-   [ ] p95/p99 latency
-   [ ] Event-loop monitoring
-   [ ] DB pool metrics

------------------------------------------------------------------------

# 126. Final Express.js Learning Order

If you are preparing for SDE interviews, study in this order:

``` text
1. Express basics
       ↓
2. Routing
       ↓
3. Middleware
       ↓
4. Request/Response
       ↓
5. Error handling
       ↓
6. Router architecture
       ↓
7. Authentication/Authorization
       ↓
8. Validation
       ↓
9. REST API design
       ↓
10. Database integration
       ↓
11. Redis/Caching
       ↓
12. Rate limiting
       ↓
13. Security
       ↓
14. Testing
       ↓
15. Streams/uploads
       ↓
16. Graceful shutdown
       ↓
17. Observability
       ↓
18. Scaling
       ↓
19. Queues/background jobs
       ↓
20. SDE2 reliability patterns
```

------------------------------------------------------------------------

# 127. The 20 Highest-Priority Express Topics

If your interview is close, master these first:

1.  **Middleware**
2.  **Middleware order**
3.  **`next()`**
4.  **Routing**
5.  **`express.Router()`**
6.  **`req.params/query/body`**
7.  **Response/status codes**
8.  **Centralized error handling**
9.  **Async error handling**
10. **Authentication**
11. **Authorization**
12. **JWT vs sessions**
13. **Validation**
14. **CORS + security**
15. **Rate limiting**
16. **Database connection pooling**
17. **Caching**
18. **Graceful shutdown**
19. **Horizontal scaling**
20. **SDE2 reliability: timeout/retry/idempotency/circuit breaker**

> **Final interview rule:** Don't stop at "Express provides middleware."
> Be able to explain **how middleware flows through the request
> pipeline, what happens when `next()` is called, how errors propagate,
> how the application scales across instances, and how you would make
> the service secure and reliable in production.**
