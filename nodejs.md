# Node.js Interview Guide --- SDE1 / SDE2

A practical, interview-oriented Node.js handbook covering fundamentals
through advanced backend engineering topics.

> **Interview strategy:** Answer with **definition → how it works → code
> example → production use/trade-off**. For SDE2 questions, always
> discuss failure modes, scalability, observability, security, and
> trade-offs.

------------------------------------------------------------------------

# 1. Node.js Fundamentals

## 1.1 What is Node.js?

Node.js is a JavaScript runtime built on the V8 JavaScript engine. It
allows JavaScript to run outside the browser and provides APIs for
networking, files, processes, streams, timers, and more.

``` js
const http = require("node:http");

const server = http.createServer((req, res) => {
  res.end("Hello Node.js");
});

server.listen(3000);
```

### Why is Node.js popular for backend development?

-   Non-blocking I/O
-   Good concurrency for I/O-heavy workloads
-   Same language across frontend and backend
-   Large ecosystem
-   Excellent support for HTTP, networking, and real-time applications

### Important interview point

Node.js is not "single-threaded for everything." JavaScript execution in
the main thread is single-threaded, while the runtime can use the
operating system and worker threads/libuv facilities for asynchronous
work.

------------------------------------------------------------------------

# 2. Node.js Architecture

## 2.1 Explain the Node.js architecture.

A useful simplified model is:

``` text
                 Node.js Application
                         |
                  JavaScript Code
                         |
                    V8 Engine
                         |
                    Call Stack
                         |
                 Node.js Runtime
                         |
        +----------------+----------------+
        |                                 |
      libuv                         Node.js APIs
        |                                 |
   Event Loop                    fs, net, http, etc.
        |
   OS / Thread Pool
```

The important pieces are:

-   **V8** --- executes JavaScript
-   **Call stack** --- executes synchronous JavaScript
-   **libuv** --- provides the event loop and async I/O infrastructure
-   **Node APIs** --- expose filesystem, networking, streams, crypto,
    etc.
-   **OS** --- handles many network operations
-   **Worker pool** --- handles certain operations that should not
    execute directly on the main event-loop thread

------------------------------------------------------------------------

## 2.2 Why is Node.js good for I/O-heavy applications?

Suppose 1,000 requests are waiting for database/network responses.

A blocking model could keep a thread waiting for each operation.

Node.js instead starts the I/O operation and allows the JavaScript
thread to continue processing other runnable work.

``` text
Request A
   |
Start DB request
   |
Continue handling other work
   |
DB completes
   |
Promise/callback becomes runnable
```

This can provide high concurrency without requiring one JavaScript
thread per connection.

------------------------------------------------------------------------

## 2.3 Is Node.js single-threaded?

The JavaScript execution model in a Node.js process has a main
thread/event loop, but Node.js is not limited to one OS thread.

Node can use:

-   OS asynchronous I/O
-   libuv's worker pool
-   Worker Threads
-   Child processes
-   Multiple Node.js processes

### Interview answer

> "Node.js executes JavaScript on a main event-loop thread, which is why
> CPU-heavy JavaScript can block requests. But the runtime can perform
> asynchronous I/O and can use worker threads or processes for
> CPU-intensive work."

------------------------------------------------------------------------

# 3. Event Loop

## 3.1 What is the Node.js event loop?

The event loop allows Node.js to perform non-blocking operations by
scheduling callbacks when asynchronous work is ready.

Simplified:

``` text
          JavaScript
              |
          Call Stack
              |
        Start async I/O
              |
       Runtime / OS / pool
              |
       Operation completes
              |
          Event Loop
              |
          Callback
              |
          Call Stack
```

Example:

``` js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

console.log("C");
```

Output:

``` text
A
C
B
```

`setTimeout(..., 0)` does not execute immediately.

------------------------------------------------------------------------

## 3.2 What are the Node.js event-loop phases?

At a high level, Node's event loop has phases including:

``` text
timers
pending callbacks
idle / prepare
poll
check
close callbacks
```

You don't need to memorize implementation details for every interview,
but know the practical distinction between:

-   Timers
-   Poll phase / I/O callbacks
-   `setImmediate`
-   Close callbacks

------------------------------------------------------------------------

## 3.3 `setTimeout` vs `setImmediate`

`setTimeout(fn, 0)` schedules a timer.

`setImmediate(fn)` schedules work for the check phase.

Inside an I/O callback, `setImmediate()` is generally designed to run
before a zero-delay timer scheduled from the same callback context.

Example:

``` js
const fs = require("node:fs");

fs.readFile(__filename, () => {
  setTimeout(() => console.log("timeout"), 0);
  setImmediate(() => console.log("immediate"));
});
```

The typical ordering here is:

``` text
immediate
timeout
```

Avoid claiming that `setTimeout(0)` and `setImmediate()` have one
universal ordering in every context.

------------------------------------------------------------------------

## 3.4 `process.nextTick()` vs Promise microtasks

Node.js has a `process.nextTick()` queue that is processed with very
high priority relative to the regular event-loop phases.

``` js
console.log("A");

process.nextTick(() => {
  console.log("B");
});

Promise.resolve().then(() => {
  console.log("C");
});

console.log("D");
```

Typical output:

``` text
A
D
B
C
```

### Interview warning

Overusing `process.nextTick()` can starve I/O because Node processes the
next-tick queue before continuing through normal event-loop work.

------------------------------------------------------------------------

# 4. Blocking vs Non-Blocking Code

## 4.1 What does blocking the event loop mean?

If JavaScript performs a long synchronous operation, other requests
cannot execute JavaScript on that main thread during that period.

Bad:

``` js
app.get("/slow", (req, res) => {
  const start = Date.now();

  while (Date.now() - start < 5000) {
    // blocks for ~5 seconds
  }

  res.send("done");
});
```

During this time, unrelated requests handled by the same process can
experience severe latency.

------------------------------------------------------------------------

## 4.2 Synchronous vs asynchronous filesystem APIs

Blocking:

``` js
const fs = require("node:fs");

const data = fs.readFileSync("large.txt", "utf8");
```

Non-blocking:

``` js
fs.readFile("large.txt", "utf8", (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

For server request paths, asynchronous APIs or streams are generally
preferred when synchronous work would create unacceptable latency.

------------------------------------------------------------------------

# 5. libuv

## 5.1 What is libuv?

libuv is a library used by Node.js to provide the event loop and
cross-platform asynchronous I/O infrastructure.

It handles or coordinates things such as:

-   Event loop
-   Timers
-   Async I/O
-   Thread pool
-   OS-specific mechanisms

### Interview point

Don't say:

> "libuv makes everything asynchronous by putting it in the thread
> pool."

That's incorrect.

Many network operations are handled by the OS asynchronously, while some
operations use libuv's worker pool.

------------------------------------------------------------------------

## 5.2 What is the libuv thread pool?

Some potentially blocking operations are offloaded to a worker pool.

Common examples include certain:

-   File-system operations
-   DNS operations
-   Crypto operations
-   Compression operations

The default pool size is commonly 4, and it can be configured using:

``` bash
UV_THREADPOOL_SIZE=8 node server.js
```

### Important

Increasing the thread pool is not automatically an optimization. You
should measure contention and understand the workload first.

------------------------------------------------------------------------

# 6. Modules

## 6.1 What is CommonJS?

CommonJS uses:

``` js
const fs = require("node:fs");

module.exports = {
  hello() {
    return "hello";
  }
};
```

Import:

``` js
const { hello } = require("./utils");
```

------------------------------------------------------------------------

## 6.2 What are ES Modules?

ES Modules use:

``` js
export function add(a, b) {
  return a + b;
}
```

and:

``` js
import { add } from "./math.js";
```

Node.js supports both CommonJS and ES Modules depending on configuration
and file/package conventions.

------------------------------------------------------------------------

## 6.3 CommonJS vs ES Modules

  Feature                         CommonJS             ES Modules
  ------------------------------- -------------------- ------------------
  Import                          `require()`          `import`
  Export                          `module.exports`     `export`
  Common in older Node projects   Yes                  Less
  Standard JS module system       No                   Yes
  Static module structure         Less explicit        Stronger
  Top-level await                 Not in classic CJS   Supported in ESM

------------------------------------------------------------------------

# 7. `package.json` and npm

## 7.1 What is `package.json`?

It describes a Node project.

``` json
{
  "name": "my-api",
  "version": "1.0.0",
  "scripts": {
    "start": "node server.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^5.0.0"
  }
}
```

It can contain:

-   Project metadata
-   Scripts
-   Dependencies
-   Dev dependencies
-   Module configuration
-   Package entry points

------------------------------------------------------------------------

## 7.2 `dependencies` vs `devDependencies`

Runtime dependencies:

``` bash
npm install express
```

Development dependencies:

``` bash
npm install -D jest
```

Typical examples:

``` text
dependencies:
express
pg
redis

devDependencies:
jest
eslint
prettier
```

------------------------------------------------------------------------

## 7.3 What is `package-lock.json`?

It records the resolved dependency tree and versions so installations
can be reproduced more consistently.

For applications, committing the lockfile is generally important.

------------------------------------------------------------------------

## 7.4 `npm install` vs `npm ci`

`npm install` resolves/updates dependencies based on package metadata
and the lockfile.

`npm ci` is intended for clean, reproducible CI installations and uses
the lockfile strictly.

Typical CI:

``` bash
npm ci
npm test
```

------------------------------------------------------------------------

# 8. Process and Environment

## 8.1 What is `process`?

`process` exposes information and controls related to the current
Node.js process.

``` js
console.log(process.pid);
console.log(process.version);
console.log(process.platform);
```

------------------------------------------------------------------------

## 8.2 How do you read environment variables?

``` js
const port = process.env.PORT || 3000;

console.log(port);
```

Run:

``` bash
PORT=8080 node server.js
```

On Windows PowerShell:

``` powershell
$env:PORT=8080
node server.js
```

Do not hard-code secrets in source code.

------------------------------------------------------------------------

## 8.3 What is `process.argv`?

It contains command-line arguments.

``` js
console.log(process.argv);
```

For:

``` bash
node app.js hello
```

you can access:

``` js
process.argv[2]; // "hello"
```

------------------------------------------------------------------------

## 8.4 `process.exit()` and exit codes

``` js
process.exit(1);
```

Exit code `0` conventionally means success; non-zero generally indicates
failure.

Avoid calling `process.exit()` unnecessarily because it can terminate
the process before pending asynchronous cleanup completes.

------------------------------------------------------------------------

# 9. Error Handling

## 9.1 How are errors handled in Node.js?

Synchronous:

``` js
try {
  JSON.parse("invalid");
} catch (error) {
  console.error(error);
}
```

Promise-based:

``` js
try {
  await doSomething();
} catch (error) {
  console.error(error);
}
```

Callback-based APIs often use error-first callbacks:

``` js
fs.readFile("file.txt", (err, data) => {
  if (err) {
    console.error(err);
    return;
  }

  console.log(data.toString());
});
```

------------------------------------------------------------------------

## 9.2 What is an error-first callback?

Node's traditional callback convention is:

``` js
callback(error, result);
```

Example:

``` js
fs.readFile("file.txt", (err, data) => {
  if (err) {
    return console.error(err);
  }

  console.log(data.toString());
});
```

If successful:

``` text
err = null
data = result
```

If failed:

``` text
err = Error
data = undefined
```

------------------------------------------------------------------------

## 9.3 What are unhandled Promise rejections?

If a Promise rejects and no appropriate rejection handler handles it,
Node can report an unhandled rejection and behavior should not be relied
upon as a substitute for explicit error handling.

Bad:

``` js
async function main() {
  throw new Error("failed");
}

main();
```

Better:

``` js
main().catch(error => {
  console.error(error);
});
```

At application boundaries, establish a consistent policy for logging,
cleanup, and process termination/restart when appropriate.

------------------------------------------------------------------------

## 9.4 Why should you not catch and ignore errors?

Bad:

``` js
try {
  await saveUser();
} catch (error) {
}
```

This can hide failures.

Better:

``` js
try {
  await saveUser();
} catch (error) {
  logger.error(error);
  throw error;
}
```

Or intentionally recover when the business requirement says failure is
acceptable.

------------------------------------------------------------------------

# 10. HTTP Server

## 10.1 How do you create an HTTP server without Express?

``` js
const http = require("node:http");

const server = http.createServer((req, res) => {
  res.writeHead(200, {
    "Content-Type": "application/json"
  });

  res.end(JSON.stringify({
    message: "Hello"
  }));
});

server.listen(3000, () => {
  console.log("Server running");
});
```

------------------------------------------------------------------------

## 10.2 What are `req` and `res`?

`req` represents the incoming HTTP request.

``` js
req.method
req.url
req.headers
```

`res` represents the outgoing response.

``` js
res.statusCode = 200;
res.setHeader("Content-Type", "text/plain");
res.end("Hello");
```

------------------------------------------------------------------------

# 11. Express.js Fundamentals

## 11.1 What is Express?

Express is a lightweight Node.js web framework commonly used for HTTP
APIs and web servers.

Example:

``` js
const express = require("express");

const app = express();

app.use(express.json());

app.get("/users/:id", (req, res) => {
  res.json({
    id: req.params.id
  });
});

app.listen(3000);
```

------------------------------------------------------------------------

## 11.2 What is middleware?

Middleware is code that participates in processing a request.

``` js
app.use((req, res, next) => {
  console.log(req.method, req.url);
  next();
});
```

`next()` passes control to the next middleware/handler.

------------------------------------------------------------------------

## 11.3 Types of middleware

Common categories:

-   Application middleware
-   Router middleware
-   Built-in middleware
-   Third-party middleware
-   Error-handling middleware

Example:

``` js
app.use(express.json());
```

------------------------------------------------------------------------

## 11.4 What is error-handling middleware?

Express error middleware conventionally has four arguments:

``` js
app.use((err, req, res, next) => {
  console.error(err);

  res.status(500).json({
    error: "Internal Server Error"
  });
});
```

The error handler should be registered after routes/middleware that can
pass errors to it.

------------------------------------------------------------------------

## 11.5 How do you structure an Express application?

A common architecture:

``` text
src/
├── routes/
├── controllers/
├── services/
├── repositories/
├── middleware/
├── models/
├── config/
├── utils/
└── app.js
```

Responsibilities:

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

This is not a mandatory folder structure. The goal is separation of
responsibilities and testability.

------------------------------------------------------------------------

# 12. REST API Design

## 12.1 How would you design a REST API?

Example:

``` text
GET    /users
GET    /users/:id
POST   /users
PATCH  /users/:id
DELETE /users/:id
```

Use appropriate status codes:

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
```

Exact API semantics should match your contract.

------------------------------------------------------------------------

## 12.2 Authentication vs authorization

Authentication asks:

> Who are you?

Authorization asks:

> What are you allowed to do?

Example:

``` text
Authentication
   ↓
JWT/session proves user identity
   ↓
Authorization
   ↓
Check whether user can delete this resource
```

------------------------------------------------------------------------

# 13. Streams

## 13.1 What is a stream?

A stream processes data incrementally instead of loading the entire
dataset into memory.

``` text
Large file
   ↓
chunk
   ↓
process
   ↓
chunk
   ↓
...
```

Types:

-   Readable
-   Writable
-   Duplex
-   Transform

------------------------------------------------------------------------

## 13.2 Readable stream example

``` js
const fs = require("node:fs");

const stream = fs.createReadStream("large-file.txt");

stream.on("data", chunk => {
  console.log("Received:", chunk.length);
});

stream.on("end", () => {
  console.log("Done");
});
```

------------------------------------------------------------------------

## 13.3 Writable stream

``` js
const fs = require("node:fs");

const output = fs.createWriteStream("output.txt");

output.write("Hello\n");
output.write("World\n");

output.end();
```

------------------------------------------------------------------------

## 13.4 Transform stream

A Transform stream can read data, transform it, and output transformed
data.

``` js
const { Transform } = require("node:stream");

const upper = new Transform({
  transform(chunk, encoding, callback) {
    callback(null, chunk.toString().toUpperCase());
  }
});
```

------------------------------------------------------------------------

# 14. Backpressure

## 14.1 What is backpressure?

Backpressure occurs when the producer generates data faster than the
consumer can process it.

``` text
Producer: 100 MB/s
Consumer: 10 MB/s
```

Without backpressure, memory buffers may grow excessively.

------------------------------------------------------------------------

## 14.2 How does Node.js handle backpressure?

When writing to a stream:

``` js
const canContinue = writable.write(chunk);

if (!canContinue) {
  // wait for "drain"
}
```

A common pattern:

``` js
if (!writable.write(chunk)) {
  await once(writable, "drain");
}
```

For stream-to-stream transfers, prefer:

``` js
readable.pipe(writable);
```

or modern pipeline utilities.

------------------------------------------------------------------------

## 14.3 What is `stream.pipeline()`?

`pipeline()` connects streams and handles completion/error propagation
more safely than manually wiring many events.

``` js
const { pipeline } = require("node:stream");
const fs = require("node:fs");

pipeline(
  fs.createReadStream("input.txt"),
  fs.createWriteStream("output.txt"),
  err => {
    if (err) console.error(err);
    else console.log("Done");
  }
);
```

------------------------------------------------------------------------

# 15. Buffers

## 15.1 What is a Buffer?

A Buffer represents raw binary data in Node.js.

``` js
const buffer = Buffer.from("hello");

console.log(buffer);
console.log(buffer.toString());
```

Useful for:

-   Files
-   TCP/network data
-   Binary protocols
-   Images
-   Encryption

------------------------------------------------------------------------

## 15.2 Buffer vs string

A string represents text.

A Buffer represents bytes.

``` js
const text = "hello";
const bytes = Buffer.from(text);

console.log(bytes);
```

Encoding matters:

``` js
Buffer.from("hello", "utf8");
```

------------------------------------------------------------------------

# 16. File System

## 16.1 Reading and writing files

``` js
const fs = require("node:fs/promises");

async function main() {
  await fs.writeFile("hello.txt", "Hello Node");
  const content = await fs.readFile("hello.txt", "utf8");

  console.log(content);
}

main();
```

------------------------------------------------------------------------

## 16.2 When should you use streams instead of `readFile`?

For small files:

``` js
await fs.readFile("config.json", "utf8");
```

is fine.

For very large files, streams avoid loading the entire file into memory:

``` js
const stream = fs.createReadStream("huge.log");
```

------------------------------------------------------------------------

# 17. EventEmitter

## 17.1 What is EventEmitter?

EventEmitter provides an event-based communication mechanism.

``` js
const EventEmitter = require("node:events");

const emitter = new EventEmitter();

emitter.on("userCreated", user => {
  console.log("Created:", user);
});

emitter.emit("userCreated", {
  id: 1,
  name: "Alice"
});
```

------------------------------------------------------------------------

## 17.2 `on` vs `once`

`on` registers a listener for every matching event.

``` js
emitter.on("event", handler);
```

`once` automatically removes the listener after the first invocation.

``` js
emitter.once("event", handler);
```

------------------------------------------------------------------------

## 17.3 When should EventEmitter be used?

Good uses:

-   Internal application events
-   Decoupling components
-   Logging/metrics hooks
-   Domain events inside a process

Don't assume EventEmitter is a durable message broker. If events must
survive process crashes or be consumed by independent services, use an
external durable system such as a message queue/streaming platform.

------------------------------------------------------------------------

# 18. Child Processes

## 18.1 What is `child_process`?

Node can create child processes to run external programs.

``` js
const { exec } = require("node:child_process");

exec("node --version", (error, stdout) => {
  if (error) {
    console.error(error);
    return;
  }

  console.log(stdout);
});
```

------------------------------------------------------------------------

## 18.2 `exec` vs `spawn`

### `exec`

Good for commands where you want buffered output.

``` js
exec("ls -la", callback);
```

But output is buffered, so very large output can be problematic.

### `spawn`

Streams stdout/stderr:

``` js
const { spawn } = require("node:child_process");

const child = spawn("node", ["script.js"]);

child.stdout.on("data", data => {
  console.log(data.toString());
});
```

------------------------------------------------------------------------

## 18.3 Why use child processes?

-   Run external programs
-   Isolate failures
-   Parallelize independent work
-   Execute CPU-heavy non-JavaScript programs

For CPU-heavy JavaScript specifically, Worker Threads are often a better
fit than spawning a separate process when shared-memory/thread-based
execution is appropriate.

------------------------------------------------------------------------

# 19. Worker Threads

## 19.1 What are Worker Threads?

Worker Threads allow JavaScript to execute in separate threads.

Useful for CPU-intensive tasks.

Main thread:

``` js
const { Worker } = require("node:worker_threads");

const worker = new Worker("./worker.js");

worker.on("message", result => {
  console.log(result);
});
```

Worker:

``` js
const { parentPort } = require("node:worker_threads");

let result = 0;

for (let i = 0; i < 1e9; i++) {
  result += i;
}

parentPort.postMessage(result);
```

The goal is to keep CPU-heavy work off the main event-loop thread.

------------------------------------------------------------------------

## 19.2 Worker Threads vs child processes

  -----------------------------------------------------------------------
  Feature                 Worker Threads          Child Processes
  ----------------------- ----------------------- -----------------------
  Execution               Separate thread         Separate process

  Memory                  Can share memory with   Separate memory space
                          mechanisms such as      
                          SharedArrayBuffer       

  Isolation               Less than process       Stronger

  CPU-heavy JS            Excellent use case      Also possible

  Process-level isolation No                      Yes
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 20. Cluster and Horizontal Scaling

## 20.1 What is Node.js clustering?

Node's cluster mechanism can run multiple Node processes that can share
server-port handling.

Conceptually:

``` text
               Load balancing
                    |
        +-----------+-----------+
        |           |           |
      Worker      Worker      Worker
      process     process     process
```

Each process has its own JavaScript heap.

Today, many production systems also scale Node processes using container
orchestration/process managers rather than relying specifically on the
built-in cluster API.

------------------------------------------------------------------------

## 20.2 Why can't one Node process use all CPU cores for JavaScript execution?

The main JavaScript execution thread is not designed to execute
JavaScript simultaneously across all cores.

To use multiple cores, you can use:

-   Worker Threads
-   Multiple processes
-   Containers/VMs
-   Orchestration/load balancing

------------------------------------------------------------------------

# 21. Database Connectivity

## 21.1 How should a Node.js service connect to a database?

Use a connection pool rather than creating a brand-new connection for
every request.

Concept:

``` text
Application
    |
Connection Pool
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

const result = await pool.query(
  "SELECT * FROM users WHERE id = $1",
  [userId]
);
```

------------------------------------------------------------------------

## 21.2 Why use a connection pool?

Opening database connections is expensive.

A pool:

-   Reuses connections
-   Limits concurrent DB connections
-   Reduces connection setup overhead
-   Provides backpressure to the application

The pool size should be tuned based on database capacity and workload
rather than made arbitrarily large.

------------------------------------------------------------------------

## 21.3 What is the N+1 query problem?

Suppose:

``` text
1 query → fetch 100 users
100 queries → fetch each user's orders
```

That's 101 queries.

Better approaches may include:

-   Joins
-   Batching
-   Data loaders
-   Bulk queries
-   Appropriate caching

------------------------------------------------------------------------

# 22. Redis and Caching

## 22.1 Why use Redis with Node.js?

Common use cases:

-   Caching
-   Sessions
-   Rate limiting
-   Distributed locks
-   Pub/sub
-   Queues
-   Counters

Example:

``` js
await redis.set(
  `user:${id}`,
  JSON.stringify(user),
  { EX: 60 }
);
```

------------------------------------------------------------------------

## 22.2 Cache-aside pattern

``` text
Request
   ↓
Check cache
   ↓
Hit? ── yes → return
   |
   no
   ↓
Database
   ↓
Store in cache
   ↓
Return
```

Example:

``` js
async function getUser(id) {
  const key = `user:${id}`;

  const cached = await redis.get(key);

  if (cached) {
    return JSON.parse(cached);
  }

  const user = await db.findUser(id);

  await redis.set(key, JSON.stringify(user), {
    EX: 60
  });

  return user;
}
```

------------------------------------------------------------------------

# 23. Rate Limiting

## 23.1 Why is rate limiting needed?

It protects services from:

-   Abuse
-   Accidental overload
-   Brute-force attacks
-   Traffic spikes
-   Expensive endpoint misuse

Common algorithms:

-   Fixed window
-   Sliding window
-   Token bucket
-   Leaky bucket

------------------------------------------------------------------------

## 23.2 Token bucket concept

``` text
        tokens
          ↓
      +---------+
      | Bucket  |
      +---------+
       ↑       ↓
    refill   request
              |
          consume token
```

A distributed Node.js API can store bucket state in Redis and use an
atomic operation/script to prevent races across multiple instances.

------------------------------------------------------------------------

# 24. Authentication

## 24.1 Session-based authentication

``` text
Login
  ↓
Server creates session
  ↓
Session ID stored in cookie
  ↓
Browser sends cookie
  ↓
Server loads session
```

Benefits:

-   Easy server-side revocation
-   Session state controlled centrally

Trade-off:

-   Requires shared session storage when horizontally scaling, unless
    sessions are otherwise coordinated.

------------------------------------------------------------------------

## 24.2 JWT authentication

Typical flow:

``` text
Login
  ↓
Server issues signed token
  ↓
Client sends token
  ↓
Server verifies signature/claims
```

Example:

``` js
const token = jwt.sign(
  { sub: user.id },
  process.env.JWT_SECRET,
  { expiresIn: "15m" }
);
```

JWTs are not automatically "better" than sessions. Choose based on
architecture, revocation requirements, token lifetime, storage, and
security model.

------------------------------------------------------------------------

## 24.3 Where should tokens be stored?

For browser applications, storage choice is a security architecture
decision.

Important considerations:

-   `HttpOnly` cookies reduce direct JavaScript access to the cookie.
-   `Secure` ensures cookies are sent over HTTPS.
-   `SameSite` helps mitigate cross-site request risks.
-   Storing long-lived sensitive tokens in `localStorage` can increase
    exposure to XSS.

There is no single storage mechanism that solves every security problem.

------------------------------------------------------------------------

# 25. Password Security

## 25.1 How should passwords be stored?

Never store plaintext passwords.

Use a password hashing algorithm designed for passwords, such as:

-   Argon2
-   bcrypt
-   scrypt

Example with bcrypt:

``` js
const bcrypt = require("bcrypt");

const hash = await bcrypt.hash(password, 12);

const valid = await bcrypt.compare(
  password,
  hash
);
```

A password hash is not encryption. You don't decrypt it; you verify a
candidate password against it.

------------------------------------------------------------------------

# 26. Security

## 26.1 How do you secure a Node.js API?

Key areas:

-   Validate input
-   Authenticate users
-   Authorize operations
-   Use parameterized queries
-   Configure security headers
-   Rate-limit sensitive endpoints
-   Protect secrets
-   Use HTTPS
-   Keep dependencies updated
-   Avoid leaking internal errors
-   Configure CORS intentionally
-   Set secure cookie attributes
-   Limit request sizes

------------------------------------------------------------------------

## 26.2 SQL injection

Bad:

``` js
const query = `
  SELECT * FROM users
  WHERE id = ${userId}
`;
```

Better:

``` js
const result = await db.query(
  "SELECT * FROM users WHERE id = $1",
  [userId]
);
```

Use parameterized queries or a properly safe query builder/ORM.

------------------------------------------------------------------------

## 26.3 NoSQL injection

Don't blindly pass user input into query operators.

Bad conceptual pattern:

``` js
db.users.findOne(req.body);
```

Validate the expected fields and types.

------------------------------------------------------------------------

## 26.4 CORS

CORS controls browser access to cross-origin responses.

Example:

``` js
app.use(cors({
  origin: "https://example.com"
}));
```

Avoid blindly enabling all origins when credentials or sensitive data
are involved.

------------------------------------------------------------------------

# 27. Input Validation

## 27.1 Why validate input?

Never assume client input is trustworthy.

Validate:

-   Types
-   Required fields
-   String length
-   Numeric ranges
-   Enum values
-   IDs
-   Nested object structure

Example using a schema-validation library conceptually:

``` js
const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(18)
});

const data = schema.parse(req.body);
```

Validation should happen near the application boundary.

------------------------------------------------------------------------

# 28. Async Patterns

## 28.1 Sequential vs concurrent operations

Sequential:

``` js
const a = await fetchA();
const b = await fetchB();
```

Concurrent when independent:

``` js
const [a, b] = await Promise.all([
  fetchA(),
  fetchB()
]);
```

This is one of the most common backend performance improvements.

------------------------------------------------------------------------

## 28.2 Limit concurrency

Don't blindly do:

``` js
await Promise.all(
  hugeArray.map(processItem)
);
```

For thousands of expensive tasks, this can overload:

-   Memory
-   Database
-   Downstream APIs
-   Connection pools

Use a concurrency limiter.

Concept:

``` text
10,000 jobs
     ↓
limit = 20
     ↓
20 active
     ↓
completed job → next job
```

------------------------------------------------------------------------

# 29. Retries

## 29.1 How should retries be implemented?

Don't retry every error.

Consider:

-   Retryable vs non-retryable errors
-   Maximum attempts
-   Timeout
-   Exponential backoff
-   Jitter
-   Idempotency
-   Cancellation

Example:

``` js
async function retry(fn, attempts = 3) {
  let lastError;

  for (let i = 0; i < attempts; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;

      if (i === attempts - 1) {
        throw error;
      }

      const delay = 100 * 2 ** i;

      await new Promise(resolve =>
        setTimeout(resolve, delay)
      );
    }
  }

  throw lastError;
}
```

Production systems often add random jitter.

------------------------------------------------------------------------

## 29.2 Why is jitter important?

Without jitter:

``` text
Service fails
    ↓
1000 clients retry
    ↓
same delay
    ↓
1000 requests at once
```

This can create a retry storm.

Jitter spreads retries over time.

------------------------------------------------------------------------

# 30. Timeouts

## 30.1 Why are timeouts important?

Without timeouts, a request may remain pending for too long and consume
resources.

Use timeouts for:

-   HTTP requests
-   Database calls
-   External services
-   Queue operations

With `fetch`, use `AbortController`:

``` js
const controller = new AbortController();

const timer = setTimeout(() => {
  controller.abort();
}, 3000);

try {
  const response = await fetch(url, {
    signal: controller.signal
  });

  return await response.json();
} finally {
  clearTimeout(timer);
}
```

------------------------------------------------------------------------

# 31. Idempotency

## 31.1 What is idempotency?

An operation is idempotent when repeating it has the same intended
effect as performing it once.

This matters because distributed systems retry requests.

Example:

``` text
POST /payments
Idempotency-Key: abc123
```

The server can store the result for `abc123` and return the same result
if the client retries.

------------------------------------------------------------------------

## 31.2 Why is idempotency important?

Imagine:

``` text
Client
  ↓
Create payment
  ↓
Server processes payment
  ↓
Network timeout
  ↓
Client retries
```

Without idempotency, the payment might be charged twice.

------------------------------------------------------------------------

# 32. Graceful Shutdown

## 32.1 What is graceful shutdown?

When a service receives SIGTERM:

``` text
SIGTERM
  ↓
Stop accepting new work
  ↓
Allow active requests to finish
  ↓
Close DB connections
  ↓
Close Redis/message connections
  ↓
Stop workers
  ↓
Exit
```

Example:

``` js
const server = app.listen(3000);

process.on("SIGTERM", async () => {
  console.log("Shutting down");

  server.close(async () => {
    await db.close();
    await redis.quit();

    process.exit(0);
  });
});
```

In production, add a shutdown deadline so a stuck request cannot prevent
termination forever.

------------------------------------------------------------------------

# 33. Health Checks

## 33.1 Liveness vs readiness

### Liveness

Answers:

> Is the process alive?

``` js
app.get("/health/live", (req, res) => {
  res.sendStatus(200);
});
```

### Readiness

Answers:

> Can this instance receive traffic?

A readiness check may verify required dependencies or application state.

``` js
app.get("/health/ready", async (req, res) => {
  const dbReady = await checkDatabase();

  if (!dbReady) {
    return res.sendStatus(503);
  }

  res.sendStatus(200);
});
```

------------------------------------------------------------------------

# 34. Logging and Observability

## 34.1 What should production logs contain?

Useful fields:

``` json
{
  "level": "error",
  "requestId": "abc123",
  "route": "/users",
  "status": 500,
  "durationMs": 142,
  "error": "Database timeout"
}
```

Avoid logging:

-   Passwords
-   Tokens
-   API secrets
-   Sensitive personal information

------------------------------------------------------------------------

## 34.2 Why use request IDs?

A request ID lets you trace a request through multiple services.

``` text
Client
  ↓ request-id: abc
API
  ↓ abc
Service A
  ↓ abc
Service B
  ↓ abc
Database/logs
```

This is extremely useful in distributed debugging.

------------------------------------------------------------------------

## 34.3 Metrics vs logs vs traces

### Logs

Detailed events.

### Metrics

Numerical measurements:

``` text
request_count
error_rate
p95_latency
CPU
memory
```

### Traces

Follow one request across components.

For SDE2, understand all three and when each is useful.

------------------------------------------------------------------------

# 35. Memory Management

## 35.1 How does Node.js manage memory?

V8 manages JavaScript heap memory and performs garbage collection.

You can inspect memory:

``` js
console.log(process.memoryUsage());
```

Common fields include:

``` text
rss
heapTotal
heapUsed
external
arrayBuffers
```

------------------------------------------------------------------------

## 35.2 What causes Node.js memory leaks?

Common causes:

-   Global arrays/maps that grow forever
-   Unbounded caches
-   Event listeners that aren't removed
-   Long-lived closures
-   Timers retaining objects
-   Large objects retained by references
-   Improper request/session state

Example:

``` js
const cache = new Map();

function add(key, value) {
  cache.set(key, value);
}
```

If the keyspace grows forever, memory can grow indefinitely.

------------------------------------------------------------------------

## 35.3 How do you diagnose memory leaks?

Approach:

1.  Observe increasing memory usage.
2.  Check whether memory returns after GC.
3.  Capture heap snapshots.
4.  Compare snapshots over time.
5.  Identify retaining paths.
6.  Fix the reference causing retention.
7.  Load-test and verify.

Tools can include Node's inspector/heap snapshots and production
monitoring.

------------------------------------------------------------------------

# 36. Event Loop Monitoring

## 36.1 How do you detect event-loop blocking?

Track event-loop delay/latency and correlate it with:

-   CPU
-   Request latency
-   Garbage collection
-   Synchronous operations

Node provides APIs in `node:perf_hooks` for monitoring event-loop
behavior.

Conceptually:

``` text
CPU spike
   +
event-loop delay spike
   +
request latency spike
        ↓
Investigate synchronous/CPU-heavy work
```

------------------------------------------------------------------------

# 37. Performance Optimization

## 37.1 How would you optimize a slow Node.js API?

Do not start by changing random code.

Process:

``` text
Measure
  ↓
Identify bottleneck
  ↓
Fix bottleneck
  ↓
Benchmark
  ↓
Monitor
```

Investigate:

-   Database latency
-   N+1 queries
-   External API latency
-   Event-loop blocking
-   CPU usage
-   Memory/GC
-   Serialization cost
-   Network payload size
-   Connection-pool saturation

------------------------------------------------------------------------

## 37.2 How can you improve API latency?

Possible improvements:

-   Database indexing
-   Query optimization
-   Caching
-   Connection pooling
-   Parallel independent I/O
-   Response compression where appropriate
-   Pagination
-   Avoid unnecessary serialization
-   Avoid blocking synchronous operations
-   Keep-alive connections
-   Reduce downstream calls

Always measure before and after.

------------------------------------------------------------------------

# 38. Pagination

## 38.1 Offset pagination

``` text
GET /users?limit=20&offset=100
```

Simple, but large offsets can become expensive depending on the
database.

------------------------------------------------------------------------

## 38.2 Cursor pagination

``` text
GET /users?limit=20&cursor=eyJpZCI6MTAwfQ
```

Concept:

``` text
First page
  ↓
last item cursor
  ↓
next page starts after cursor
```

Cursor pagination is often better for large/changing datasets.

------------------------------------------------------------------------

# 39. Caching

## 39.1 Cache-aside

``` text
Request
  ↓
Cache?
 /   \
hit   miss
 |      |
return  DB
        |
      cache
        |
      return
```

------------------------------------------------------------------------

## 39.2 Cache stampede

Suppose a popular cache entry expires:

``` text
Cache expires
     ↓
1000 requests
     ↓
1000 DB queries
```

Solutions:

-   Request coalescing
-   Locking
-   Early refresh
-   Randomized expiration
-   Stale-while-revalidate

------------------------------------------------------------------------

## 39.3 Request deduplication

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

This can prevent duplicate downstream work for concurrent requests.

------------------------------------------------------------------------

# 40. Queues and Background Jobs

## 40.1 Why use a job queue?

Move slow/non-critical work out of the request path.

Instead of:

``` text
HTTP request
  ↓
Generate report
  ↓
Send email
  ↓
Upload file
  ↓
Response
```

Use:

``` text
HTTP request
  ↓
Create job
  ↓
Return quickly
  ↓
Worker processes job
```

Examples of queue technologies include Redis-backed job systems and
dedicated message brokers.

------------------------------------------------------------------------

## 40.2 What makes a good background job system?

Consider:

-   Retries
-   Dead-letter handling
-   Idempotency
-   Visibility/timeouts
-   Concurrency limits
-   Backpressure
-   Monitoring
-   Job priority
-   Delayed jobs
-   Graceful shutdown

------------------------------------------------------------------------

# 41. Transactions

## 41.1 What is a database transaction?

A transaction groups operations into an atomic unit.

``` text
BEGIN
  update account A
  update account B
COMMIT
```

If a failure occurs:

``` text
ROLLBACK
```

Example:

``` js
await client.query("BEGIN");

try {
  await client.query(...);
  await client.query(...);

  await client.query("COMMIT");
} catch (error) {
  await client.query("ROLLBACK");
  throw error;
}
```

------------------------------------------------------------------------

# 42. Race Conditions

## 42.1 Can Node.js have race conditions?

Yes.

Single-threaded JavaScript does not eliminate logical races between
asynchronous operations.

Example:

``` js
let balance = 100;

async function withdraw(amount) {
  const current = balance;

  await someAsyncOperation();

  balance = current - amount;
}
```

Two concurrent withdrawals can read the same old balance.

Solutions may include:

-   Database transactions
-   Atomic database updates
-   Distributed locks
-   Optimistic concurrency control
-   Queues

------------------------------------------------------------------------

# 43. Distributed Locks

## 43.1 When would you need a distributed lock?

When multiple application instances must coordinate access to a shared
resource.

``` text
Server A ──┐
           ├── shared lock
Server B ──┤
Server C ──┘
```

Use carefully. Distributed locks introduce failure and lease-expiry
concerns.

Often an atomic database operation or idempotent design is preferable.

------------------------------------------------------------------------

# 44. HTTP Keep-Alive and Connection Reuse

## 44.1 Why reuse HTTP connections?

Creating a TCP/TLS connection repeatedly adds overhead.

Connection pooling/keep-alive can reduce:

-   TCP handshake overhead
-   TLS handshake overhead
-   Latency
-   CPU cost

For high-throughput Node services, use an HTTP client with sensible
connection pooling/keep-alive configuration.

------------------------------------------------------------------------

# 45. Rate Limits and Downstream Protection

## 45.1 How would you protect a downstream API?

Use:

-   Concurrency limits
-   Rate limits
-   Timeouts
-   Retries only when safe
-   Circuit breakers
-   Caching
-   Request deduplication
-   Bulkheads

------------------------------------------------------------------------

# 46. Circuit Breaker

## 46.1 What is a circuit breaker?

A circuit breaker prevents repeatedly calling an unhealthy dependency.

States:

``` text
          failure threshold
CLOSED --------------------> OPEN
  ↑                            |
  |                            | wait
  |                            ↓
  +------------------------ HALF-OPEN
          success
```

Typical behavior:

### Closed

Requests flow normally.

### Open

Requests fail fast.

### Half-open

Allow limited test requests.

Useful for protecting both the caller and an unhealthy downstream
service.

------------------------------------------------------------------------

# 47. Bulkheads

## 47.1 What is the bulkhead pattern?

Isolate resources so one failing dependency doesn't consume everything.

Example:

``` text
Service
 |
 +-- Payment pool
 |
 +-- Email pool
 |
 +-- Search pool
```

If email becomes slow, it should not consume every worker/connection and
block payment traffic.

------------------------------------------------------------------------

# 48. Graceful Degradation

## 48.1 What is graceful degradation?

When a dependency fails, provide a reduced but useful service.

Example:

``` text
Recommendation service unavailable
        ↓
Return product details
        ↓
Skip recommendations
```

This is often better than failing the entire request.

------------------------------------------------------------------------

# 49. Node.js Streams + HTTP

## 49.1 How would you serve a large file?

Avoid:

``` js
const data = await fs.readFile("10GB.iso");
res.end(data);
```

Prefer streaming:

``` js
const fs = require("node:fs");

app.get("/download", (req, res) => {
  const stream = fs.createReadStream("10GB.iso");

  stream.on("error", error => {
    console.error(error);
    res.destroy(error);
  });

  stream.pipe(res);
});
```

This reduces peak application memory usage.

------------------------------------------------------------------------

# 50. File Uploads

## 50.1 How would you handle large file uploads?

Don't necessarily buffer the entire upload in memory.

Use streaming/multipart processing and enforce:

-   Maximum size
-   Allowed file types
-   Authentication
-   Timeouts
-   Storage limits
-   Malware scanning where required

Architecture:

``` text
Client
  ↓
Node upload endpoint
  ↓
Stream
  ↓
Object storage
```

------------------------------------------------------------------------

# 51. WebSockets

## 51.1 When would you use WebSockets?

When the server and client need a persistent, bidirectional connection.

Examples:

-   Chat
-   Live collaboration
-   Real-time dashboards
-   Multiplayer applications
-   Live notifications

Architecture:

``` text
Browser ←────────→ Node server
       persistent connection
```

For multiple Node instances, shared state/pub-sub coordination is often
required.

------------------------------------------------------------------------

# 52. Server-Sent Events

## 52.1 What are SSE?

Server-Sent Events provide a long-lived HTTP connection where the server
sends events to the browser.

Good for:

-   Notifications
-   Live progress
-   Streaming updates

Unlike WebSockets, communication is primarily server → client.

------------------------------------------------------------------------

# 53. WebSockets vs SSE

  -----------------------------------------------------------------------
  Feature                 WebSocket               SSE
  ----------------------- ----------------------- -----------------------
  Direction               Bidirectional           Server → client

  Protocol                WebSocket               HTTP

  Browser reconnect       Application-managed     Built-in browser
  support                                         EventSource behavior

  Good for chat           Yes                     Less suitable

  Good for server updates Yes                     Excellent
  -----------------------------------------------------------------------

Choose based on communication requirements.

------------------------------------------------------------------------

# 54. Testing Node.js

## 54.1 Unit tests

Test a small unit in isolation.

``` js
function add(a, b) {
  return a + b;
}
```

Test:

``` js
expect(add(2, 3)).toBe(5);
```

------------------------------------------------------------------------

## 54.2 Integration tests

Test components together:

``` text
API
 ↓
Service
 ↓
Database
```

Useful for verifying actual integration behavior.

------------------------------------------------------------------------

## 54.3 API tests

Test:

-   Status codes
-   Response body
-   Validation
-   Authentication
-   Authorization
-   Error handling
-   Database effects

------------------------------------------------------------------------

## 54.4 What should you mock?

Mock external systems when isolation is useful:

-   Payment provider
-   Email service
-   External HTTP API

Don't mock everything. Excessive mocking can make tests tightly coupled
to implementation details.

------------------------------------------------------------------------

# 55. Configuration Management

## 55.1 How should Node applications manage configuration?

Separate configuration from code.

``` js
const config = {
  port: Number(process.env.PORT || 3000),
  databaseUrl: process.env.DATABASE_URL
};
```

Validate required configuration at startup:

``` js
if (!process.env.DATABASE_URL) {
  throw new Error("DATABASE_URL is required");
}
```

Fail fast for invalid critical configuration.

------------------------------------------------------------------------

# 56. Secrets Management

Never do:

``` js
const password = "super-secret-password";
```

Use:

-   Environment variables for simple deployments
-   Secret managers for production
-   Managed identity mechanisms where available

Never commit secrets to Git.

If a secret is leaked, rotate it.

------------------------------------------------------------------------

# 57. Dependency Security

## 57.1 How do you keep Node dependencies secure?

Use:

``` bash
npm audit
```

Also:

-   Keep dependencies updated
-   Review dependency changes
-   Lock versions appropriately
-   Remove unused packages
-   Use automated dependency/security scanning
-   Avoid untrusted packages

A vulnerability in a transitive dependency can also affect your
application.

------------------------------------------------------------------------

# 58. SDE1 Must-Know Node.js Questions

Before an SDE1 interview, master:

1.  What is Node.js?
2.  Why is Node.js useful for I/O-heavy applications?
3.  Is Node.js single-threaded?
4.  Explain the event loop.
5.  Explain libuv.
6.  What blocks the event loop?
7.  `setTimeout` vs `setImmediate`
8.  `process.nextTick`
9.  Promise microtasks
10. CommonJS vs ESM
11. `package.json`
12. `package-lock.json`
13. npm install vs npm ci
14. Environment variables
15. HTTP server
16. Express middleware
17. Error middleware
18. REST API design
19. Streams
20. Buffers
21. EventEmitter
22. File system APIs
23. Async/await
24. Promise.all
25. Database connection pooling
26. Redis caching
27. Authentication vs authorization
28. JWT vs sessions
29. Input validation
30. SQL injection
31. CORS
32. Graceful shutdown
33. Basic testing
34. Logging
35. Rate limiting

------------------------------------------------------------------------

# 59. SDE2 Must-Know Node.js Questions

For SDE2, be able to reason about:

1.  Event-loop latency
2.  CPU-bound work
3.  Worker Threads
4.  Child processes
5.  Horizontal scaling
6.  Connection-pool sizing
7.  Backpressure
8.  Streaming large data
9.  Cache stampede
10. Request deduplication
11. Retry storms
12. Exponential backoff + jitter
13. Idempotency
14. Timeouts
15. Circuit breakers
16. Bulkheads
17. Graceful degradation
18. Distributed locks
19. Race conditions
20. Database transactions
21. N+1 queries
22. Cursor pagination
23. Queue-based architecture
24. Dead-letter handling
25. Memory leaks
26. Heap snapshots
27. Event-loop monitoring
28. CPU profiling
29. Observability
30. Request tracing
31. Graceful shutdown
32. Readiness/liveness
33. API security
34. Dependency security
35. Service architecture

------------------------------------------------------------------------

# 60. High-Value Coding Questions

## Q1. Implement debounce

``` js
function debounce(fn, delay) {
  let timer;

  return function (...args) {
    clearTimeout(timer);

    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}
```

Know how to explain:

-   Closure
-   Timer
-   Cancellation
-   `this`
-   Arguments

------------------------------------------------------------------------

## Q2. Implement concurrency limiting

``` js
async function runWithLimit(tasks, limit) {
  const results = new Array(tasks.length);
  let next = 0;

  async function worker() {
    while (true) {
      const index = next++;

      if (index >= tasks.length) {
        return;
      }

      results[index] = await tasks[index]();
    }
  }

  const workers = Array.from(
    { length: Math.min(limit, tasks.length) },
    () => worker()
  );

  await Promise.all(workers);

  return results;
}
```

Follow-ups:

-   What happens if one task fails?
-   How do you cancel remaining tasks?
-   How do you add retries?
-   How do you enforce a global rate limit?

------------------------------------------------------------------------

## Q3. Implement retry with backoff

``` js
async function retry(fn, attempts = 3) {
  for (let i = 0; i < attempts; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === attempts - 1) {
        throw error;
      }

      const delay = 100 * 2 ** i;

      await new Promise(resolve =>
        setTimeout(resolve, delay)
      );
    }
  }
}
```

Production follow-ups:

-   Retry only transient failures
-   Add jitter
-   Respect deadlines
-   Support cancellation
-   Make operations idempotent

------------------------------------------------------------------------

## Q4. Implement a simple TTL cache

``` js
class TTLCache {
  constructor() {
    this.cache = new Map();
  }

  set(key, value, ttlMs) {
    const expiresAt = Date.now() + ttlMs;

    this.cache.set(key, {
      value,
      expiresAt
    });
  }

  get(key) {
    const item = this.cache.get(key);

    if (!item) return undefined;

    if (Date.now() >= item.expiresAt) {
      this.cache.delete(key);
      return undefined;
    }

    return item.value;
  }
}
```

SDE2 follow-ups:

-   What about maximum cache size?
-   LRU eviction?
-   Distributed cache?
-   Cache stampede?
-   Serialization?
-   Memory leak?
-   Thread/process consistency?

------------------------------------------------------------------------

## Q5. Implement request deduplication

``` js
const inFlight = new Map();

async function getData(key) {
  if (inFlight.has(key)) {
    return inFlight.get(key);
  }

  const promise = expensiveOperation(key)
    .finally(() => {
      inFlight.delete(key);
    });

  inFlight.set(key, promise);

  return promise;
}
```

This prevents duplicate concurrent work for the same key within the
process.

------------------------------------------------------------------------

# 61. Output-Based Interview Questions

## Question 1

``` js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

Promise.resolve().then(() => {
  console.log("C");
});

console.log("D");
```

Output:

``` text
A
D
C
B
```

Reason:

``` text
Synchronous
  ↓
Microtask
  ↓
Timer task
```

------------------------------------------------------------------------

## Question 2

``` js
console.log("A");

process.nextTick(() => {
  console.log("B");
});

Promise.resolve().then(() => {
  console.log("C");
});

setImmediate(() => {
  console.log("D");
});

console.log("E");
```

A typical Node ordering is:

``` text
A
E
B
C
D
```

The exact ordering of timers/immediates can depend on context, so
explain the relevant queue/phase rather than memorizing a universal
sequence.

------------------------------------------------------------------------

## Question 3

``` js
const fs = require("node:fs");

fs.readFile(__filename, () => {
  setTimeout(() => console.log("timeout"), 0);
  setImmediate(() => console.log("immediate"));
});
```

Typical output:

``` text
immediate
timeout
```

Reason: inside the I/O callback, `setImmediate()` runs in the check
phase after poll, while the timer is handled in the timers phase of a
later iteration.

------------------------------------------------------------------------

# 62. Production Architecture

## 62.1 Design a scalable Node.js API

A reasonable starting architecture:

``` text
                   Load Balancer
                        |
          +-------------+-------------+
          |             |             |
       Node API      Node API      Node API
          |             |             |
          +-------------+-------------+
                        |
                 Redis / Cache
                        |
                    Database
                        |
                Background Queue
                        |
                     Workers
```

Important concerns:

-   Stateless API servers
-   Connection pooling
-   Caching
-   Rate limiting
-   Timeouts
-   Retries
-   Idempotency
-   Queue-based background work
-   Observability
-   Graceful shutdown
-   Horizontal scaling

------------------------------------------------------------------------

# 63. How to Debug a Slow Node.js Service

Use this investigation sequence:

``` text
1. Is latency actually high?
          ↓
2. Which endpoint?
          ↓
3. p50 / p95 / p99?
          ↓
4. CPU high?
          ↓
5. Event-loop delay high?
          ↓
6. Database slow?
          ↓
7. External API slow?
          ↓
8. Connection pool saturated?
          ↓
9. GC/memory pressure?
          ↓
10. Profile and fix bottleneck
```

Don't say:

> "I'll add caching."

First identify whether the bottleneck is actually cacheable.

------------------------------------------------------------------------

# 64. How to Handle a Traffic Spike

Suppose traffic suddenly increases 10x.

Think:

``` text
Traffic spike
    ↓
Load balancer
    ↓
Horizontal scaling
    ↓
Rate limiting
    ↓
Cache
    ↓
DB connection pool
    ↓
Database capacity
```

Protect downstream systems using:

-   Rate limiting
-   Queues
-   Backpressure
-   Concurrency limits
-   Caching
-   Load shedding
-   Autoscaling

------------------------------------------------------------------------

# 65. What happens when the database goes down?

A good SDE2 answer:

``` text
DB unavailable
     ↓
Requests fail/timeout
     ↓
Timeout prevents resource exhaustion
     ↓
Retry only transient failures
     ↓
Exponential backoff + jitter
     ↓
Circuit breaker may open
     ↓
Graceful degradation if possible
     ↓
Alert + observability
```

Don't blindly retry every request.

------------------------------------------------------------------------

# 66. How to Design a Reliable External API Call

A robust flow:

``` text
Request
  ↓
Validate input
  ↓
Set deadline/timeout
  ↓
Call dependency
  ↓
Success? ── yes → return
  |
  no
  ↓
Retryable?
 /      \
no       yes
 |        |
fail   backoff
          |
       retry
          |
       max attempts
          |
       circuit breaker
```

Add idempotency where the operation can create side effects.

------------------------------------------------------------------------

# 67. Node.js Best Practices

## Code

-   Prefer asynchronous APIs in request paths
-   Keep functions small and testable
-   Validate external input
-   Handle errors explicitly
-   Avoid global mutable state
-   Use parameterized DB queries
-   Use environment/configuration management
-   Keep dependencies minimal

## Performance

-   Avoid event-loop blocking
-   Use streams for large data
-   Reuse DB/HTTP connections
-   Parallelize independent I/O
-   Limit concurrency
-   Cache carefully
-   Profile before optimizing

## Reliability

-   Timeouts
-   Retries with backoff
-   Idempotency
-   Graceful shutdown
-   Health checks
-   Circuit breakers where appropriate
-   Queue-based background work

## Security

-   HTTPS
-   Authentication
-   Authorization
-   Input validation
-   Rate limiting
-   Secure cookies
-   Secret management
-   Dependency scanning
-   Safe error messages

------------------------------------------------------------------------

# 68. Rapid-Fire Node.js Interview Questions

Be able to answer these in 30--60 seconds each:

### Fundamentals

1.  What is Node.js?
2.  What is V8?
3.  What is libuv?
4.  Why is Node good for I/O-heavy applications?
5.  Is Node.js single-threaded?
6.  What blocks the event loop?
7.  What is the event loop?
8.  What are event-loop phases?
9.  `process.nextTick` vs Promise microtasks?
10. `setImmediate` vs `setTimeout`?

### Backend

11. What is middleware?
12. How does Express middleware work?
13. How do you handle errors?
14. How do you validate requests?
15. How do you structure a Node API?
16. How do you implement authentication?
17. Session vs JWT?
18. How do you secure cookies?
19. How do you prevent SQL injection?
20. What is CORS?

### Performance

21. How do you avoid blocking?
22. When do you use Worker Threads?
23. Worker Threads vs child processes?
24. Why use streams?
25. What is backpressure?
26. How do you optimize database access?
27. Why use connection pools?
28. How do you implement caching?
29. What is cache stampede?
30. How do you limit concurrency?

### Distributed systems

31. What is idempotency?
32. Why are retries dangerous?
33. Why use exponential backoff?
34. What is jitter?
35. What is a circuit breaker?
36. What are bulkheads?
37. How do you handle dependency failure?
38. How do you design graceful shutdown?
39. How do you handle duplicate requests?
40. How do you scale Node horizontally?

------------------------------------------------------------------------

# 69. SDE2 Scenario Questions

## Scenario 1: API latency suddenly becomes 5x

Discuss:

``` text
Metrics
 ↓
Endpoint identification
 ↓
p95/p99
 ↓
DB latency
 ↓
External API latency
 ↓
Event-loop delay
 ↓
CPU/memory/GC
 ↓
Connection pool
 ↓
Recent deployment
```

Then fix the actual bottleneck and validate the change.

------------------------------------------------------------------------

## Scenario 2: Node process memory keeps increasing

Investigate:

``` text
Heap usage
 ↓
GC behavior
 ↓
Heap snapshots
 ↓
Retaining paths
 ↓
Caches
 ↓
Listeners
 ↓
Timers
 ↓
Global state
```

Then reproduce under controlled load and verify memory stabilizes.

------------------------------------------------------------------------

## Scenario 3: Downstream service is failing

Do not immediately retry infinitely.

Use:

``` text
Timeout
+
Retry only transient failures
+
Exponential backoff
+
Jitter
+
Circuit breaker
+
Fallback/degradation
+
Metrics/alerts
```

------------------------------------------------------------------------

## Scenario 4: Database is overloaded

Possible causes:

-   Too many connections
-   Slow queries
-   Missing indexes
-   N+1 queries
-   Excessive retries
-   Traffic spike

Possible mitigations:

-   Fix queries/indexes
-   Tune pool size
-   Cache
-   Rate limit
-   Queue work
-   Read replicas where appropriate
-   Reduce unnecessary queries

------------------------------------------------------------------------

# 70. Final Node.js Interview Roadmap

## Level 1 --- SDE1 Core

Master:

``` text
Node.js
  ↓
V8
  ↓
Event Loop
  ↓
Async/Await
  ↓
Promises
  ↓
Express
  ↓
REST APIs
  ↓
Middleware
  ↓
Database
  ↓
Redis
  ↓
Authentication
  ↓
Streams
```

## Level 2 --- Strong SDE1

Add:

``` text
Connection pooling
Caching
Rate limiting
Error handling
Validation
Testing
Logging
Graceful shutdown
Security
```

## Level 3 --- SDE2

Master:

``` text
Event-loop performance
Worker Threads
Streams/backpressure
Concurrency control
Retries
Timeouts
Idempotency
Circuit breakers
Bulkheads
Cache stampede
Request deduplication
Queues
Transactions
Race conditions
Observability
Memory leaks
Horizontal scaling
```

## Level 4 --- SDE2 System Design

Be able to design:

``` text
Scalable REST API
Real-time service
File upload/download service
Rate limiter
Job processing system
Notification service
Payment workflow
Caching layer
API gateway
Distributed worker system
```

------------------------------------------------------------------------

# 71. The 20 Topics to Master First

If the interview is close, prioritize these:

1.  **Node.js architecture**
2.  **Event loop**
3.  **libuv**
4.  **Blocking vs non-blocking code**
5.  **Promises + async/await**
6.  **`process.nextTick` and microtasks**
7.  **Express middleware**
8.  **Error handling**
9.  **Streams**
10. **Backpressure**
11. **Database connection pooling**
12. **Caching**
13. **Redis**
14. **Rate limiting**
15. **Worker Threads**
16. **Retries + exponential backoff + jitter**
17. **Timeouts + cancellation**
18. **Idempotency**
19. **Graceful shutdown**
20. **Observability + performance debugging**

> **Final interview rule:** Don't just explain what Node.js feature
> does. Explain **why you would use it, what can go wrong, and what
> trade-off it introduces**. That distinction is especially important
> when moving from SDE1 to SDE2.
