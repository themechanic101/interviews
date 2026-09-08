# JavaScript Interview Guide --- SDE1 / SDE2

A practical JavaScript interview handbook covering fundamentals,
execution model, asynchronous JavaScript, browser APIs,
objects/prototypes, functional programming, modern JavaScript, Node.js
concepts, performance, and SDE2-level design/debugging topics.

> **How to use this:** For interviews, answer in this order:
> **definition → key behavior → example → practical use/trade-off**. For
> output questions, first predict the output, then explain the execution
> model.

------------------------------------------------------------------------

# 1. JavaScript Fundamentals

## 1.1 What is JavaScript?

JavaScript is a high-level, dynamically typed programming language
primarily used to build interactive applications. It runs in browsers
and also in server-side runtimes such as Node.js.

Important characteristics:

-   Dynamically typed
-   Garbage collected
-   Prototype-based object model
-   First-class functions
-   Supports object-oriented, functional, and imperative programming
-   Single-threaded JavaScript execution model, with asynchronous
    capabilities provided by the runtime

``` js
const name = "Alice";
console.log(`Hello ${name}`);
```

**Interview point:** JavaScript itself is the language; things such as
the DOM, `fetch`, timers, and Node.js APIs are provided by the host
runtime.

------------------------------------------------------------------------

## 1.2 What are JavaScript's data types?

JavaScript has 8 primitive types:

-   `string`
-   `number`
-   `bigint`
-   `boolean`
-   `undefined`
-   `null`
-   `symbol`
-   `object` (non-primitive)

``` js
const name = "Alice";       // string
const age = 25;             // number
const huge = 123n;          // bigint
const active = true;        // boolean
let value;                  // undefined
const empty = null;         // null
const id = Symbol("id");    // symbol
const user = { name };      // object
```

**Interview trap:**

``` js
typeof null; // "object" — historical language quirk
typeof [];   // "object"
typeof NaN;  // "number"
```

------------------------------------------------------------------------

## 1.3 Primitive vs reference/object values

Primitive values are immutable values. Objects are mutable entities
accessed through references.

``` js
let a = 10;
let b = a;
b = 20;

console.log(a); // 10
```

With objects:

``` js
const a = { count: 1 };
const b = a;

b.count = 2;

console.log(a.count); // 2
```

`a` and `b` refer to the same object.

**Interview point:** Saying "objects are passed by reference" is a
common simplification. More precisely, JavaScript is pass-by-value; when
the value is an object, the copied value is a reference to that object.

------------------------------------------------------------------------

## 1.4 `null` vs `undefined`

`undefined` commonly means a value is missing or has not been assigned.
`null` is an intentional "no value" value.

``` js
let x;
console.log(x); // undefined

let selectedUser = null;
```

Comparison:

``` js
null == undefined;  // true
null === undefined; // false
```

------------------------------------------------------------------------

## 1.5 What is `NaN`?

`NaN` means "Not a Number", but its JavaScript type is `number`.

``` js
const result = Number("hello");

console.log(result);       // NaN
console.log(typeof result); // "number"
```

Important:

``` js
NaN === NaN;        // false
Number.isNaN(NaN);  // true
```

Prefer `Number.isNaN()` when you specifically want to test for `NaN`.

------------------------------------------------------------------------

## 1.6 What are truthy and falsy values?

Falsy values include:

``` text
false
0
-0
0n
""
null
undefined
NaN
```

Almost everything else is truthy, including:

``` js
[]
{}
```

Example:

``` js
if ([]) {
  console.log("runs");
}
```

------------------------------------------------------------------------

## 1.7 What is type coercion?

Type coercion is conversion between types during an operation.

``` js
"5" + 2; // "52"
"5" - 2; // 3
"5" * 2; // 10
```

JavaScript converted values according to the operation.

Explicit conversion is clearer:

``` js
const input = "42";
const number = Number(input);
```

**Interview advice:** Understand coercion well, but prefer explicit
conversion in production code when clarity matters.

------------------------------------------------------------------------

## 1.8 `==` vs `===`

`==` allows type coercion; `===` checks value and type without
performing that loose equality conversion.

``` js
5 == "5";  // true
5 === "5"; // false

0 == false;  // true
0 === false; // false
```

**Preferred:** `===` in most production code.

------------------------------------------------------------------------

## 1.9 What is `Object.is()`?

`Object.is()` performs SameValue comparison and differs from `===` in a
few edge cases.

``` js
Object.is(NaN, NaN); // true
NaN === NaN;         // false

Object.is(0, -0); // false
0 === -0;         // true
```

------------------------------------------------------------------------

## 1.10 What are template literals?

Template literals use backticks and support interpolation and multiline
strings.

``` js
const name = "Alice";
const age = 25;

const message = `My name is ${name} and I am ${age}.`;
```

They can contain expressions:

``` js
const total = `Total: ${10 + 20}`;
```

------------------------------------------------------------------------

## 1.11 `let`, `const`, and `var`

### `var`

-   Function scoped
-   Can be redeclared
-   Hoisted and initialized to `undefined`

### `let`

-   Block scoped
-   Cannot be redeclared in the same scope
-   Has a Temporal Dead Zone

### `const`

-   Block scoped
-   Must be initialized
-   Cannot be reassigned

``` js
let count = 1;
count = 2;

const user = { name: "Alice" };
user.name = "Bob"; // allowed

// user = {}; // TypeError
```

`const` prevents reassignment of the binding, not mutation of the
object.

------------------------------------------------------------------------

## 1.12 What is hoisting?

Declarations are processed before normal execution of the corresponding
scope.

``` js
console.log(x); // undefined
var x = 10;
```

Conceptually:

``` js
var x;
console.log(x);
x = 10;
```

Function declarations can be called before their declaration:

``` js
sayHello();

function sayHello() {
  console.log("Hello");
}
```

`let` and `const` are hoisted but cannot be accessed before
initialization because of the Temporal Dead Zone.

------------------------------------------------------------------------

## 1.13 What is the Temporal Dead Zone?

The TDZ is the period between entering a scope and execution reaching a
`let` or `const` declaration.

``` js
console.log(x); // ReferenceError
let x = 10;
```

------------------------------------------------------------------------

## 1.14 What is scope?

Scope determines where a variable can be accessed.

Main types:

-   Global scope
-   Function scope
-   Block scope
-   Lexical scope

``` js
const globalValue = 10;

function test() {
  const localValue = 20;

  if (true) {
    const blockValue = 30;
    console.log(globalValue, localValue, blockValue);
  }
}
```

------------------------------------------------------------------------

## 1.15 What is lexical scope?

Lexical scope means variable visibility is determined by where code is
written.

``` js
const x = 10;

function outer() {
  const y = 20;

  function inner() {
    console.log(x, y);
  }

  inner();
}
```

`inner` can access variables from its lexical outer environments.

------------------------------------------------------------------------

# 2. Functions, Closures, and `this`

## 2.1 What are first-class functions?

Functions are values in JavaScript. They can be:

-   Assigned to variables
-   Passed as arguments
-   Returned from functions
-   Stored in objects/arrays

``` js
function add(a, b) {
  return a + b;
}

const fn = add;
console.log(fn(2, 3));

function createMultiplier(x) {
  return n => n * x;
}

const double = createMultiplier(2);
console.log(double(5)); // 10
```

------------------------------------------------------------------------

## 2.2 What is a callback?

A callback is a function supplied to another function to be invoked
later or during an operation.

``` js
function processUser(user, callback) {
  callback(user.name);
}

processUser({ name: "Alice" }, name => {
  console.log(name);
});
```

Callbacks are heavily used in event handling and asynchronous
programming.

------------------------------------------------------------------------

## 2.3 What is a closure?

A closure is created when a function retains access to variables from
its lexical environment even after the outer function has returned.

``` js
function counter() {
  let count = 0;

  return function increment() {
    count++;
    return count;
  };
}

const increment = counter();

console.log(increment()); // 1
console.log(increment()); // 2
console.log(increment()); // 3
```

### Practical uses

-   Private state
-   Function factories
-   Memoization
-   Callbacks
-   Event handlers
-   Maintaining state between calls

------------------------------------------------------------------------

## 2.4 What is the `this` keyword?

`this` is determined by how a function is invoked, with important
differences for arrow functions.

``` js
const user = {
  name: "Alice",
  greet() {
    console.log(this.name);
  }
};

user.greet(); // Alice
```

Here `this` is the object used as the receiver of the call.

------------------------------------------------------------------------

## 2.5 How does `this` behave in arrow functions?

Arrow functions do not have their own `this`. They lexically capture it
from the surrounding scope.

``` js
const user = {
  name: "Alice",

  greet() {
    setTimeout(() => {
      console.log(this.name);
    }, 100);
  }
};

user.greet(); // Alice
```

The arrow callback retains `this` from `greet`.

**Interview trap:**

``` js
const user = {
  name: "Alice",
  greet: () => console.log(this.name)
};
```

The arrow function does not get `this` from `user`.

------------------------------------------------------------------------

## 2.6 `call`, `apply`, and `bind`

They allow explicit control over `this` for regular functions.

``` js
function greet(city) {
  console.log(`${this.name} from ${city}`);
}

const user = { name: "Alice" };

greet.call(user, "Delhi");
greet.apply(user, ["Delhi"]);

const bound = greet.bind(user);
bound("Delhi");
```

Difference:

-   `call`: invokes immediately, arguments separately
-   `apply`: invokes immediately, arguments as an array-like value
-   `bind`: returns a new function with bound `this`

------------------------------------------------------------------------

## 2.7 Regular functions vs arrow functions

  Feature                  Regular   Arrow
  ------------------------ --------- -------
  Own `this`               Yes       No
  Own `arguments`          Yes       No
  Constructor with `new`   Yes       No
  Lexical `this`           No        Yes
  Concise syntax           No        Yes

Use regular methods when you need dynamic `this`; arrow functions are
excellent for callbacks and lexical `this`.

------------------------------------------------------------------------

## 2.8 What is an IIFE?

Immediately Invoked Function Expression.

``` js
(function () {
  const secret = "hidden";
  console.log(secret);
})();
```

Historically used to create private scope before modules became
standard.

------------------------------------------------------------------------

## 2.9 What are default parameters?

``` js
function greet(name = "Guest") {
  return `Hello ${name}`;
}

console.log(greet()); // Hello Guest
```

The default is used when the argument is `undefined`.

------------------------------------------------------------------------

## 2.10 What is the rest parameter?

It collects remaining arguments into an array.

``` js
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

console.log(sum(1, 2, 3, 4)); // 10
```

------------------------------------------------------------------------

## 2.11 What is the spread operator?

Spread expands an iterable or object into individual
elements/properties.

``` js
const a = [1, 2];
const b = [...a, 3, 4];

const user = { name: "Alice" };
const updated = { ...user, age: 25 };
```

**Important:** spread creates a shallow copy.

------------------------------------------------------------------------

# 3. Objects, Prototypes, and Classes

## 3.1 How are objects created?

``` js
const user = {
  name: "Alice",
  age: 25
};

const obj = new Object();

const another = Object.create(null);
```

You can also use constructor functions or classes.

------------------------------------------------------------------------

## 3.2 What is a prototype?

Every ordinary JavaScript object has a prototype from which it can
inherit properties and methods.

``` js
const user = {
  name: "Alice"
};

console.log(Object.getPrototypeOf(user));
```

Property lookup checks the object and then its prototype chain.

------------------------------------------------------------------------

## 3.3 What is the prototype chain?

If a property isn't found on an object, JavaScript looks up its
prototype, then that prototype's prototype, and so on.

``` js
const parent = {
  greet() {
    console.log("Hello");
  }
};

const child = Object.create(parent);

child.greet(); // Hello
```

Lookup:

``` text
child
  ↓
parent
  ↓
Object.prototype
  ↓
null
```

------------------------------------------------------------------------

## 3.4 `__proto__` vs `prototype`

This is a frequent interview question.

-   `prototype` is a property commonly found on constructor functions.
-   `__proto__` is a legacy accessor exposing an object's prototype.

``` js
function User() {}

console.log(User.prototype);

const user = new User();

console.log(Object.getPrototypeOf(user) === User.prototype); // true
```

Prefer `Object.getPrototypeOf()` / `Object.setPrototypeOf()` for
explicit prototype operations rather than relying on `__proto__`.

------------------------------------------------------------------------

## 3.5 How does `new` work?

For:

``` js
function User(name) {
  this.name = name;
}

const user = new User("Alice");
```

Conceptually, `new`:

1.  Creates a new object.
2.  Links it to `User.prototype`.
3.  Calls `User` with `this` bound to the new object.
4.  Returns the object unless the constructor explicitly returns another
    object.

------------------------------------------------------------------------

## 3.6 What are constructor functions?

Before `class` syntax, constructor functions were commonly used.

``` js
function User(name) {
  this.name = name;
}

User.prototype.greet = function () {
  return `Hi ${this.name}`;
};

const user = new User("Alice");
console.log(user.greet());
```

------------------------------------------------------------------------

## 3.7 How do JavaScript classes work?

Classes provide syntax over JavaScript's prototype-based object model.

``` js
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hi ${this.name}`;
  }
}

const user = new User("Alice");
console.log(user.greet());
```

Methods are placed on the prototype rather than recreated as separate
own properties for every instance.

------------------------------------------------------------------------

## 3.8 What is inheritance in JavaScript?

``` js
class Animal {
  speak() {
    console.log("Animal sound");
  }
}

class Dog extends Animal {
  speak() {
    console.log("Woof");
  }
}

const dog = new Dog();
dog.speak();
```

`Dog` inherits from `Animal` and overrides `speak`.

------------------------------------------------------------------------

## 3.9 What is `super`?

`super` accesses the parent class.

``` js
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }
}
```

------------------------------------------------------------------------

## 3.10 Static methods

Static methods belong to the class itself rather than its instances.

``` js
class MathUtil {
  static add(a, b) {
    return a + b;
  }
}

console.log(MathUtil.add(2, 3));
// MathUtil instances don't have add()
```

------------------------------------------------------------------------

## 3.11 Private class fields

JavaScript supports private fields using `#`.

``` js
class BankAccount {
  #balance = 0;

  deposit(amount) {
    this.#balance += amount;
  }

  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount();
account.deposit(100);

console.log(account.getBalance()); // 100
```

------------------------------------------------------------------------

# 4. Equality, Copying, and Object Utilities

## 4.1 Shallow copy vs deep copy

Shallow copy copies only the top-level structure.

``` js
const original = {
  name: "Alice",
  address: {
    city: "Delhi"
  }
};

const copy = { ...original };

copy.address.city = "Mumbai";

console.log(original.address.city); // Mumbai
```

The nested object is still shared.

------------------------------------------------------------------------

## 4.2 How can you deep clone an object?

Modern JavaScript provides:

``` js
const copy = structuredClone(original);
```

This is generally preferable to JSON serialization when supported and
when you need the types it supports.

JSON cloning:

``` js
const copy = JSON.parse(JSON.stringify(original));
```

has limitations: it loses or changes certain values/types such as
`undefined`, functions, `BigInt`, and some object types.

------------------------------------------------------------------------

## 4.3 `Object.keys`, `Object.values`, `Object.entries`

``` js
const user = {
  name: "Alice",
  age: 25
};

Object.keys(user);
// ["name", "age"]

Object.values(user);
// ["Alice", 25]

Object.entries(user);
// [["name", "Alice"], ["age", 25]]
```

------------------------------------------------------------------------

## 4.4 What is destructuring?

Destructuring extracts values from arrays or objects.

``` js
const user = {
  name: "Alice",
  age: 25
};

const { name, age } = user;
```

Array:

``` js
const [first, second] = [10, 20];
```

------------------------------------------------------------------------

## 4.5 Optional chaining

Optional chaining avoids errors when an intermediate value is `null` or
`undefined`.

``` js
const city = user?.address?.city;
```

Instead of:

``` js
const city =
  user &&
  user.address &&
  user.address.city;
```

------------------------------------------------------------------------

## 4.6 Nullish coalescing

`??` uses the right side only when the left side is `null` or
`undefined`.

``` js
const count = value ?? 0;
```

Compare:

``` js
0 || 10;  // 10
0 ?? 10;  // 0
```

Use `??` when `0`, `false`, or `""` are valid values.

------------------------------------------------------------------------

## 4.7 What is immutability?

Immutability means not modifying an existing value/object directly;
instead, create a new value.

``` js
const user = {
  name: "Alice",
  age: 25
};

const updatedUser = {
  ...user,
  age: 26
};
```

Useful in state management because changes become easier to reason
about.

------------------------------------------------------------------------

# 5. Arrays and Common Methods

## 5.1 `map`, `filter`, `reduce`

### `map`

Transforms every item.

``` js
const numbers = [1, 2, 3];

const doubled = numbers.map(x => x * 2);
// [2, 4, 6]
```

### `filter`

Keeps matching items.

``` js
const even = numbers.filter(x => x % 2 === 0);
// [2]
```

### `reduce`

Combines items into one result.

``` js
const sum = numbers.reduce((acc, x) => acc + x, 0);
// 6
```

------------------------------------------------------------------------

## 5.2 `find` vs `filter`

`find` returns the first matching element.

``` js
users.find(user => user.id === 10);
```

`filter` returns all matching elements.

``` js
users.filter(user => user.active);
```

------------------------------------------------------------------------

## 5.3 `some` vs `every`

``` js
numbers.some(x => x > 10);
```

Returns true if at least one matches.

``` js
numbers.every(x => x > 0);
```

Returns true if all match.

------------------------------------------------------------------------

## 5.4 `forEach` vs `map`

`forEach` is for side effects and does not produce a transformed array.

``` js
numbers.forEach(x => console.log(x));
```

`map` creates a new array.

``` js
const doubled = numbers.map(x => x * 2);
```

------------------------------------------------------------------------

## 5.5 `slice` vs `splice`

`slice` does not mutate the original array.

``` js
const a = [1, 2, 3, 4];
const b = a.slice(1, 3);

console.log(b); // [2, 3]
```

`splice` mutates the array.

``` js
const a = [1, 2, 3, 4];

a.splice(1, 2);

console.log(a); // [1, 4]
```

------------------------------------------------------------------------

# 6. Asynchronous JavaScript

## 6.1 Why is JavaScript called single-threaded?

JavaScript execution traditionally uses one main call stack for
JavaScript code in a given agent.

That doesn't mean the entire runtime can do only one thing. Browsers and
Node.js provide APIs and mechanisms that perform asynchronous I/O,
timers, networking, etc.

The key idea:

``` text
JavaScript execution
       ↓
   Call Stack
       ↓
Runtime handles async work
       ↓
Callbacks become runnable
       ↓
Event Loop schedules them
```

------------------------------------------------------------------------

## 6.2 What is the event loop?

The event loop coordinates execution of queued asynchronous callbacks
with the JavaScript call stack.

``` js
console.log("Start");

setTimeout(() => {
  console.log("Timer");
}, 0);

console.log("End");
```

Output:

``` text
Start
End
Timer
```

`setTimeout(..., 0)` does not mean "run immediately."

------------------------------------------------------------------------

## 6.3 What is a Promise?

A Promise represents the eventual result of an asynchronous operation.

States:

``` text
pending → fulfilled
pending → rejected
```

Example:

``` js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Success");
  }, 1000);
});

promise
  .then(value => console.log(value))
  .catch(error => console.error(error));
```

------------------------------------------------------------------------

## 6.4 What are `async` and `await`?

`async/await` is syntax built around Promises.

``` js
async function getUser() {
  try {
    const response = await fetch("/api/user");
    const user = await response.json();
    return user;
  } catch (error) {
    console.error(error);
  }
}
```

Important:

``` js
async function test() {
  return 10;
}
```

`test()` returns a Promise that fulfills with `10`.

------------------------------------------------------------------------

## 6.5 Does `await` block JavaScript?

No. `await` pauses the execution of the current async function until the
awaited Promise settles. It does not block the entire JavaScript
runtime.

``` js
async function test() {
  console.log("A");

  await Promise.resolve();

  console.log("B");
}

test();
console.log("C");

// A
// C
// B
```

------------------------------------------------------------------------

## 6.6 What is the microtask queue?

Promise reactions such as `.then()`, `.catch()`, and `.finally()` are
scheduled as microtasks.

``` js
console.log("A");

Promise.resolve().then(() => {
  console.log("B");
});

console.log("C");
```

Output:

``` text
A
C
B
```

------------------------------------------------------------------------

## 6.7 Microtask queue vs task queue

Typical mental model:

``` text
Run current synchronous JavaScript
          ↓
Drain microtasks
          ↓
Run a task/callback
          ↓
Drain microtasks
          ↓
Continue
```

Example:

``` js
console.log("1");

setTimeout(() => console.log("2"), 0);

Promise.resolve().then(() => console.log("3"));

console.log("4");

// 1
// 4
// 3
// 2
```

------------------------------------------------------------------------

## 6.8 What is callback hell?

Nested callbacks can make asynchronous control flow difficult to read
and maintain.

``` js
getUser(user => {
  getOrders(user, orders => {
    getPayment(orders, payment => {
      sendEmail(payment, () => {
        console.log("done");
      });
    });
  });
});
```

Promises and `async/await` provide clearer composition.

------------------------------------------------------------------------

## 6.9 Promise chaining

``` js
fetchUser()
  .then(user => fetchOrders(user.id))
  .then(orders => fetchPayment(orders[0].id))
  .then(payment => console.log(payment))
  .catch(error => console.error(error));
```

A `.then()` handler can return another Promise, causing the next step to
wait for it.

------------------------------------------------------------------------

## 6.10 `Promise.all`

Runs multiple independent Promises concurrently and rejects if any input
Promise rejects.

``` js
const [user, orders] = await Promise.all([
  fetchUser(),
  fetchOrders()
]);
```

Use it when operations are independent.

------------------------------------------------------------------------

## 6.11 `Promise.allSettled`

Waits for all input Promises and reports each outcome.

``` js
const results = await Promise.allSettled([
  fetchUser(),
  fetchOrders(),
  fetchRecommendations()
]);
```

Useful when partial failure is acceptable.

------------------------------------------------------------------------

## 6.12 `Promise.race`

Settles when the first input Promise settles, whether fulfilled or
rejected.

``` js
const result = await Promise.race([
  fetchData(),
  timeout(3000)
]);
```

Often used for timeouts, although a production timeout wrapper should
also consider cancellation.

------------------------------------------------------------------------

## 6.13 `Promise.any`

Fulfills when the first input Promise fulfills. It rejects only if all
inputs reject.

``` js
const result = await Promise.any([
  fetchFromServerA(),
  fetchFromServerB(),
  fetchFromServerC()
]);
```

Useful for redundant sources where the first successful response is
enough.

------------------------------------------------------------------------

## 6.14 Sequential vs concurrent async operations

Bad if independent:

``` js
const a = await fetchA();
const b = await fetchB();
```

This waits for A before starting B.

Better:

``` js
const [a, b] = await Promise.all([
  fetchA(),
  fetchB()
]);
```

If B depends on A, sequential execution is correct:

``` js
const user = await fetchUser();
const orders = await fetchOrders(user.id);
```

------------------------------------------------------------------------

## 6.15 How do you handle async errors?

With `try/catch`:

``` js
async function load() {
  try {
    const data = await fetchData();
    return data;
  } catch (error) {
    console.error(error);
    throw error;
  }
}
```

At an application boundary, decide whether to recover, transform, log,
or propagate the error.

------------------------------------------------------------------------

# 7. Advanced Event Loop Questions

## 7.1 Predict the output

``` js
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve().then(() => console.log("C"));

queueMicrotask(() => console.log("D"));

console.log("E");
```

Output:

``` text
A
E
C
D
B
```

Reason:

1.  Synchronous code: A, E
2.  Microtasks in insertion order: C, D
3.  Timer task: B

------------------------------------------------------------------------

## 7.2 What happens if microtasks keep adding microtasks?

The runtime drains the microtask queue before moving to the next task,
so an endlessly replenished microtask queue can starve timers and other
tasks.

``` js
function loop() {
  queueMicrotask(loop);
}

loop();
```

This is an example of why runaway microtask scheduling is dangerous.

------------------------------------------------------------------------

## 7.3 Is `setTimeout(fn, 0)` guaranteed to run after exactly 0 ms?

No. It specifies a minimum delay before the callback becomes eligible to
run. Actual execution depends on the runtime and event loop.

------------------------------------------------------------------------

# 8. DOM and Browser APIs

## 8.1 What is the DOM?

The DOM (Document Object Model) is a tree representation of an HTML
document exposed to JavaScript.

``` js
const heading = document.querySelector("h1");

heading.textContent = "Hello";
```

------------------------------------------------------------------------

## 8.2 How do you select DOM elements?

Common methods:

``` js
document.getElementById("app");

document.querySelector(".item");

document.querySelectorAll(".item");

document.getElementsByClassName("item");

document.getElementsByTagName("div");
```

Modern code commonly uses `querySelector` and `querySelectorAll`.

------------------------------------------------------------------------

## 8.3 `innerHTML` vs `textContent`

``` js
element.textContent = userInput;
```

sets text.

``` js
element.innerHTML = userInput;
```

parses the input as HTML.

For untrusted input, `textContent` is safer because it doesn't interpret
the string as HTML.

------------------------------------------------------------------------

## 8.4 What is event propagation?

Events move through phases:

``` text
Capturing → Target → Bubbling
```

Example:

``` js
parent.addEventListener("click", handler);
child.addEventListener("click", handler);
```

A click on `child` can reach the parent during bubbling.

------------------------------------------------------------------------

## 8.5 Capturing vs bubbling

``` js
parent.addEventListener("click", handler, true);
```

The `true`/capture option registers for the capturing phase.

More explicit:

``` js
parent.addEventListener("click", handler, {
  capture: true
});
```

Without capture, event listeners normally run during bubbling.

------------------------------------------------------------------------

## 8.6 What is event delegation?

Attach one listener to a parent and handle events from its children.

``` js
document.querySelector("#list").addEventListener("click", event => {
  const button = event.target.closest("button");

  if (!button) return;

  console.log("Clicked:", button.dataset.id);
});
```

Benefits:

-   Fewer event listeners
-   Works well with dynamically added elements
-   Centralized handling

------------------------------------------------------------------------

## 8.7 `preventDefault()` vs `stopPropagation()`

`preventDefault()` prevents the browser's default action.

``` js
form.addEventListener("submit", event => {
  event.preventDefault();
});
```

`stopPropagation()` stops propagation of the event through the DOM.

``` js
button.addEventListener("click", event => {
  event.stopPropagation();
});
```

They solve different problems.

------------------------------------------------------------------------

## 8.8 What are Web APIs?

Browser runtimes expose APIs such as:

-   DOM APIs
-   Timers
-   Fetch
-   Web Storage
-   Geolocation
-   WebSockets
-   Web Workers

These are runtime APIs, not simply features of the core ECMAScript
language.

------------------------------------------------------------------------

## 8.9 Browser storage

### `localStorage`

Persistent key-value storage associated with an origin.

``` js
localStorage.setItem("theme", "dark");

const theme = localStorage.getItem("theme");
```

### `sessionStorage`

Similar API, but data is scoped to a browser tab/session.

### Cookies

Often used for HTTP state such as session identifiers. Cookies can be
configured with security attributes such as `HttpOnly`, `Secure`, and
`SameSite`.

------------------------------------------------------------------------

## 8.10 What is CORS?

CORS (Cross-Origin Resource Sharing) is a browser security mechanism
controlling whether a web page can make certain cross-origin requests
and access their responses.

The server communicates its policy using HTTP headers such as:

``` text
Access-Control-Allow-Origin
```

**Interview point:** CORS is primarily enforced by browsers; it is not a
server-to-server restriction.

------------------------------------------------------------------------

# 9. Modules

## 9.1 What are ES modules?

ES modules provide a standard way to split code into files with explicit
imports and exports.

``` js
// math.js
export function add(a, b) {
  return a + b;
}
```

``` js
// app.js
import { add } from "./math.js";

console.log(add(2, 3));
```

Benefits:

-   Explicit dependencies
-   Better organization
-   Encapsulation
-   Static structure useful to tooling/bundlers

------------------------------------------------------------------------

## 9.2 Named vs default exports

Named:

``` js
export const x = 10;
export function add(a, b) {
  return a + b;
}
```

Import:

``` js
import { x, add } from "./module.js";
```

Default:

``` js
export default function add(a, b) {
  return a + b;
}
```

Import:

``` js
import add from "./module.js";
```

A module can have one default export but multiple named exports.

------------------------------------------------------------------------

## 9.3 CommonJS vs ES Modules

CommonJS:

``` js
const fs = require("fs");

module.exports = {
  value: 10
};
```

ES Modules:

``` js
import fs from "node:fs";

export const value = 10;
```

Node.js supports both systems, subject to project/module configuration.

------------------------------------------------------------------------

# 10. Error Handling

## 10.1 `try/catch/finally`

``` js
try {
  riskyOperation();
} catch (error) {
  console.error(error);
} finally {
  console.log("Cleanup");
}
```

`finally` runs after the try/catch flow, generally regardless of success
or failure.

------------------------------------------------------------------------

## 10.2 Throwing custom errors

``` js
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError";
  }
}

throw new ValidationError("Invalid email");
```

Useful for distinguishing error categories.

------------------------------------------------------------------------

## 10.3 Why shouldn't you swallow errors?

Avoid:

``` js
try {
  await save();
} catch (error) {
  // nothing
}
```

This hides failures and makes debugging difficult.

Better:

``` js
try {
  await save();
} catch (error) {
  logger.error(error);
  throw error;
}
```

or deliberately recover if the business logic permits it.

------------------------------------------------------------------------

# 11. Functional JavaScript

## 11.1 What is a pure function?

A pure function:

1.  Produces the same output for the same inputs.
2.  Has no observable side effects.

``` js
function add(a, b) {
  return a + b;
}
```

Impure:

``` js
let total = 0;

function addToTotal(x) {
  total += x;
}
```

------------------------------------------------------------------------

## 11.2 What is a higher-order function?

A higher-order function takes functions as arguments or returns a
function.

``` js
function multiplyBy(x) {
  return n => n * x;
}

const triple = multiplyBy(3);

console.log(triple(4)); // 12
```

------------------------------------------------------------------------

## 11.3 What is currying?

Currying transforms a function with multiple arguments into a sequence
of single-argument functions.

``` js
const add = a => b => a + b;

console.log(add(2)(3)); // 5
```

Useful for composition and creating specialized functions.

------------------------------------------------------------------------

## 11.4 What is memoization?

Memoization caches function results for previously seen inputs.

``` js
function memoize(fn) {
  const cache = new Map();

  return function (x) {
    if (cache.has(x)) {
      return cache.get(x);
    }

    const result = fn(x);
    cache.set(x, result);

    return result;
  };
}

const square = memoize(x => x * x);

console.log(square(5)); // computes
console.log(square(5)); // cache
```

Trade-off: memory usage increases.

------------------------------------------------------------------------

# 12. Debouncing and Throttling

## 12.1 What is debouncing?

Debouncing delays execution until activity stops for a specified period.

Common use:

-   Search input
-   Autosave
-   Resize handling

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

const search = debounce(query => {
  console.log("API call:", query);
}, 300);
```

If the user types continuously, previous timers are canceled.

------------------------------------------------------------------------

## 12.2 What is throttling?

Throttling limits execution to at most once per interval.

Common use:

-   Scroll events
-   Mouse movement
-   Continuous UI events

Conceptually:

``` text
Events:    x x x x x x x x x
Throttle:  x     x     x
```

Debounce waits for silence; throttle limits frequency.

------------------------------------------------------------------------

# 13. Memory and Garbage Collection

## 13.1 How does garbage collection work?

JavaScript engines automatically reclaim memory that is no longer
reachable.

A simplified model is reachability:

``` text
GC Roots
  ↓
reachable objects
  ↓
unreachable objects → eligible for collection
```

Modern engines use sophisticated tracing and generational techniques;
don't assume garbage collection happens immediately when an object
becomes unreachable.

------------------------------------------------------------------------

## 13.2 What causes memory leaks?

Common causes:

-   Unremoved event listeners
-   Long-lived timers
-   Global references
-   Growing caches
-   Closures retaining large objects
-   Detached DOM trees retained by JavaScript

Example:

``` js
const cache = [];

function addData(data) {
  cache.push(data);
}
```

If `cache` grows forever, memory usage can grow indefinitely.

------------------------------------------------------------------------

## 13.3 How do you avoid event-listener leaks?

Use cleanup when appropriate:

``` js
function setup() {
  const handler = () => console.log("click");

  button.addEventListener("click", handler);

  return () => {
    button.removeEventListener("click", handler);
  };
}
```

The same function reference is required for reliable removal.

------------------------------------------------------------------------

# 14. WeakMap and WeakSet

## 14.1 What is WeakMap?

`WeakMap` stores key-value pairs where keys must be objects and are
weakly held.

``` js
const metadata = new WeakMap();

let user = {};

metadata.set(user, {
  lastSeen: Date.now()
});

console.log(metadata.get(user));
```

If `user` becomes otherwise unreachable, its WeakMap entry does not by
itself keep it alive.

Useful for object-associated metadata.

------------------------------------------------------------------------

## 14.2 WeakSet

Stores objects weakly and supports membership tracking.

``` js
const processed = new WeakSet();

const request = {};

processed.add(request);

console.log(processed.has(request)); // true
```

------------------------------------------------------------------------

# 15. Map and Set

## 15.1 `Map` vs object

`Map` is designed as a general key-value collection.

``` js
const map = new Map();

map.set("userId", 101);
map.set(42, "answer");

console.log(map.get("userId"));
```

Advantages include arbitrary key types and useful collection APIs.

------------------------------------------------------------------------

## 15.2 Set

`Set` stores unique values.

``` js
const ids = new Set([1, 2, 2, 3]);

console.log([...ids]); // [1, 2, 3]
```

Common deduplication:

``` js
const unique = [...new Set(numbers)];
```

------------------------------------------------------------------------

# 16. Iterators and Generators

## 16.1 What is an iterator?

An iterator follows a protocol involving a `next()` method returning
objects such as:

``` js
{ value: 1, done: false }
```

Arrays, strings, Maps, and Sets are iterable.

------------------------------------------------------------------------

## 16.2 What is a generator?

A generator function can pause and resume execution.

``` js
function* numbers() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = numbers();

console.log(gen.next());
console.log(gen.next());
```

Generators are useful for lazy sequences and custom iteration.

------------------------------------------------------------------------

# 17. Symbols

## 17.1 What is a Symbol?

A Symbol is a primitive used to create unique identifiers.

``` js
const id1 = Symbol("id");
const id2 = Symbol("id");

console.log(id1 === id2); // false
```

Useful for object keys that should avoid accidental name collisions.

``` js
const id = Symbol("id");

const user = {
  name: "Alice",
  [id]: 123
};
```

------------------------------------------------------------------------

# 18. Proxy and Reflect

## 18.1 What is Proxy?

A Proxy lets you intercept operations on an object.

``` js
const user = {
  name: "Alice"
};

const proxy = new Proxy(user, {
  get(target, property) {
    console.log("Reading:", property);
    return Reflect.get(target, property);
  }
});

console.log(proxy.name);
```

Possible uses:

-   Validation
-   Logging
-   Reactive systems
-   Access control abstractions

Use carefully because proxies can make behavior less obvious and can
have performance implications.

------------------------------------------------------------------------

## 18.2 What is Reflect?

`Reflect` provides methods for standard object operations.

``` js
Reflect.get(user, "name");
Reflect.set(user, "name", "Bob");
Reflect.has(user, "name");
```

Proxy traps often delegate to `Reflect`.

------------------------------------------------------------------------

# 19. Async Iteration

## 19.1 What is `for await...of`?

It consumes async iterables.

``` js
async function* stream() {
  yield 1;
  yield 2;
  yield 3;
}

for await (const value of stream()) {
  console.log(value);
}
```

Useful when processing asynchronous streams of data.

------------------------------------------------------------------------

# 20. AbortController and Cancellation

## 20.1 How can you cancel a fetch request?

``` js
const controller = new AbortController();

fetch("/api/data", {
  signal: controller.signal
});

controller.abort();
```

The request is aborted and the Promise rejects with an abort-related
error.

Useful for:

-   Search requests becoming stale
-   Component cleanup
-   Request timeouts
-   User cancellation

------------------------------------------------------------------------

# 21. Browser Performance

## 21.1 What causes layout thrashing?

Repeatedly reading layout information and then modifying the DOM can
cause unnecessary layout recalculation.

Bad pattern:

``` js
for (const item of items) {
  item.style.width = box.offsetWidth + "px";
}
```

Better approach: batch reads and writes where possible.

``` text
Read measurements
       ↓
Calculate
       ↓
Apply writes
```

------------------------------------------------------------------------

## 21.2 What is event delegation useful for performance?

Instead of:

``` js
items.forEach(item => {
  item.addEventListener("click", handler);
});
```

use one parent handler:

``` js
list.addEventListener("click", event => {
  const item = event.target.closest(".item");
  if (item) handler(item);
});
```

This can reduce listener count, especially for large/dynamic lists.

------------------------------------------------------------------------

# 22. SDE1 Output-Based Questions

## 22.1 `var` loop closure

``` js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

Output:

``` text
3
3
3
```

`var` is function-scoped, so all callbacks close over the same binding.

With `let`:

``` js
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

Output:

``` text
0
1
2
```

Each iteration has an appropriate per-iteration binding.

------------------------------------------------------------------------

## 22.2 Promise and timer

``` js
console.log(1);

setTimeout(() => console.log(2), 0);

Promise.resolve().then(() => console.log(3));

console.log(4);
```

Output:

``` text
1
4
3
2
```

------------------------------------------------------------------------

## 22.3 `this` with method extraction

``` js
const user = {
  name: "Alice",
  greet() {
    console.log(this.name);
  }
};

user.greet();

const fn = user.greet;
fn();
```

The first call uses `user` as the receiver. The second is a plain
function call, so `this` is different and in strict mode is `undefined`.

------------------------------------------------------------------------

## 22.4 Shallow-copy trap

``` js
const a = {
  nested: {
    value: 1
  }
};

const b = { ...a };

b.nested.value = 2;

console.log(a.nested.value);
```

Output:

``` text
2
```

Because spread is shallow.

------------------------------------------------------------------------

# 23. Node.js Interview Fundamentals

## 23.1 What is Node.js?

Node.js is a JavaScript runtime built on the V8 engine, with APIs for
server-side and systems programming.

It is well suited to I/O-heavy applications because asynchronous I/O can
be handled without blocking the JavaScript execution thread.

Common uses:

-   APIs
-   Real-time applications
-   Backend services
-   CLI tools
-   Streaming systems
-   Proxies/gateways

------------------------------------------------------------------------

## 23.2 Why is Node.js good for I/O-heavy workloads?

Consider:

``` text
Request
  ↓
Start DB/network operation
  ↓
JavaScript thread can continue handling other work
  ↓
I/O completes
  ↓
Callback/Promise continuation runs
```

This model allows a server to handle many concurrent I/O operations
efficiently.

**Important:** Node.js is not magically faster for CPU-heavy work.
CPU-bound JavaScript can block the main event loop.

------------------------------------------------------------------------

## 23.3 What blocks the Node.js event loop?

Examples:

``` js
while (true) {}
```

or expensive synchronous computation:

``` js
crypto.pbkdf2Sync(...);
```

or processing a huge dataset synchronously.

A blocked event loop delays unrelated requests and callbacks.

------------------------------------------------------------------------

## 23.4 How do you handle CPU-heavy work in Node.js?

Options include:

-   Worker threads
-   Separate processes
-   Job queues
-   External services
-   Horizontal scaling

Worker threads example concept:

``` js
const worker = new Worker("./worker.js");
```

The key architecture is to keep the main event loop responsive.

------------------------------------------------------------------------

# 24. Node.js Modules and Package Management

## 24.1 What is `package.json`?

It describes a Node.js project, including metadata, scripts,
dependencies, and module configuration.

Example:

``` json
{
  "name": "my-api",
  "scripts": {
    "start": "node server.js",
    "test": "jest"
  }
}
```

------------------------------------------------------------------------

## 24.2 Dependencies vs devDependencies

`dependencies` are required by the application at runtime.

`devDependencies` are primarily needed for development, testing,
linting, building, etc.

------------------------------------------------------------------------

## 24.3 What is semantic versioning?

Version format:

``` text
MAJOR.MINOR.PATCH
```

Example:

``` text
2.4.1
```

Generally:

-   MAJOR: breaking changes
-   MINOR: backward-compatible features
-   PATCH: backward-compatible fixes

Package range syntax matters, so don't assume every semver range has
identical upgrade behavior.

------------------------------------------------------------------------

# 25. Node.js EventEmitter

## 25.1 What is EventEmitter?

Node.js provides an event-based abstraction.

``` js
const EventEmitter = require("node:events");

const emitter = new EventEmitter();

emitter.on("login", user => {
  console.log(`${user} logged in`);
});

emitter.emit("login", "Alice");
```

Useful for decoupled event-driven components.

------------------------------------------------------------------------

# 26. Streams and Buffers

## 26.1 What is a Buffer?

A Buffer represents raw binary data in Node.js.

``` js
const buffer = Buffer.from("hello");

console.log(buffer);
console.log(buffer.toString());
```

Useful for files, network packets, binary protocols, etc.

------------------------------------------------------------------------

## 26.2 What are streams?

Streams process data incrementally rather than loading everything into
memory at once.

Conceptually:

``` text
Large file
   ↓
Chunk
   ↓
Process
   ↓
Next chunk
   ↓
...
```

Types include:

-   Readable
-   Writable
-   Duplex
-   Transform

------------------------------------------------------------------------

## 26.3 Why are streams important?

Suppose you need to send a 10 GB file.

Naive approach:

``` text
Read entire 10 GB
       ↓
Memory
       ↓
Send
```

A stream can do:

``` text
Read chunk → send chunk → read next chunk
```

This reduces memory pressure and supports backpressure.

------------------------------------------------------------------------

# 27. Backpressure

## 27.1 What is backpressure?

Backpressure occurs when a producer generates data faster than the
consumer can process it.

Example:

``` text
Producer: 100 MB/s
Consumer: 10 MB/s
```

Without backpressure, buffers can grow and memory usage can become
dangerous.

Streams provide mechanisms for coordinating producer and consumer
speeds.

------------------------------------------------------------------------

# 28. HTTP/API JavaScript Questions

## 28.1 How do you make an API request?

``` js
async function getUsers() {
  const response = await fetch("/api/users");

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  return response.json();
}
```

**Interview point:** `fetch()` does not reject merely because the HTTP
status is 404/500. You generally need to check `response.ok` or
`response.status`.

------------------------------------------------------------------------

## 28.2 GET vs POST vs PUT vs PATCH vs DELETE

Typical semantics:

-   GET: retrieve
-   POST: create/process
-   PUT: replace/update representation
-   PATCH: partial update
-   DELETE: remove

Know that actual API behavior depends on the server contract.

------------------------------------------------------------------------

## 28.3 What is idempotency?

An operation is idempotent if repeating it has the same intended effect
as performing it once.

For example, a PUT that sets:

``` json
{
  "status": "active"
}
```

can be designed to be idempotent.

This is important in distributed systems because requests may be
retried.

------------------------------------------------------------------------

# 29. Security Questions

## 29.1 What is XSS?

Cross-Site Scripting occurs when untrusted input is interpreted as
executable content in a user's browser.

Dangerous pattern:

``` js
element.innerHTML = userInput;
```

Safer for plain text:

``` js
element.textContent = userInput;
```

Other defenses include proper output encoding, CSP, safe templating, and
avoiding unsafe HTML sinks.

------------------------------------------------------------------------

## 29.2 What is CSRF?

Cross-Site Request Forgery tricks a user's browser into making an
unwanted authenticated request.

Common defenses include:

-   SameSite cookies
-   CSRF tokens
-   Origin/Referer validation where appropriate

------------------------------------------------------------------------

## 29.3 What is prototype pollution?

Prototype pollution occurs when attacker-controlled input modifies
object prototypes in unsafe application code.

Avoid unsafe dynamic property assignment and carefully
validate/allowlist keys when merging untrusted objects.

------------------------------------------------------------------------

# 30. SDE2-Level JavaScript Concepts

## 30.1 How would you design a retry mechanism?

A production retry mechanism should consider:

-   Which errors are retryable?
-   Maximum retry count
-   Exponential backoff
-   Jitter
-   Timeout
-   Cancellation
-   Idempotency

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
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }

  throw lastError;
}
```

For distributed systems, add jitter:

``` text
delay = exponentialBackoff + randomJitter
```

This avoids many clients retrying simultaneously.

------------------------------------------------------------------------

## 30.2 How would you implement a timeout?

``` js
function timeout(ms) {
  return new Promise((_, reject) => {
    setTimeout(() => {
      reject(new Error("Timeout"));
    }, ms);
  });
}

const result = await Promise.race([
  fetchData(),
  timeout(3000)
]);
```

For APIs that support cancellation, prefer combining timeout behavior
with `AbortController` so the underlying operation is actually canceled
rather than merely ignoring its eventual result.

------------------------------------------------------------------------

## 30.3 How would you limit concurrency?

Suppose you have 10,000 URLs. Don't necessarily do:

``` js
await Promise.all(urls.map(fetchUrl));
```

This can create too much concurrent work.

A concurrency limiter keeps only N operations active.

Concept:

``` text
10,000 jobs
    ↓
Concurrency = 10
    ↓
10 running
    ↓
completed job starts next
```

This protects the application and downstream services.

------------------------------------------------------------------------

## 30.4 How would you cache expensive operations?

Use a cache such as `Map` for process-local caching:

``` js
const cache = new Map();

async function getUser(id) {
  if (cache.has(id)) {
    return cache.get(id);
  }

  const user = await fetchUser(id);
  cache.set(id, user);

  return user;
}
```

Production considerations:

-   TTL
-   Maximum size
-   Eviction policy
-   Cache invalidation
-   Stale data
-   Distributed cache such as Redis
-   Stampede protection

------------------------------------------------------------------------

## 30.5 What is a cache stampede?

If a popular cached item expires, many requests can simultaneously query
the database.

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
-   Randomized TTL
-   Stale-while-revalidate

------------------------------------------------------------------------

## 30.6 What is request deduplication?

If multiple callers request the same expensive operation simultaneously,
share one in-flight Promise.

``` js
const inFlight = new Map();

function getUser(id) {
  if (inFlight.has(id)) {
    return inFlight.get(id);
  }

  const promise = fetchUser(id)
    .finally(() => inFlight.delete(id));

  inFlight.set(id, promise);

  return promise;
}
```

Now concurrent requests for the same ID can share the same underlying
operation.

------------------------------------------------------------------------

# 31. Advanced JavaScript Architecture

## 31.1 How would you structure a large Node.js service?

A common separation:

``` text
src/
├── routes/
├── controllers/
├── services/
├── repositories/
├── models/
├── middleware/
├── utils/
├── config/
└── app.js
```

Typical responsibility:

-   Routes: map HTTP endpoints
-   Controllers: translate HTTP input/output
-   Services: business logic
-   Repositories: persistence/data access
-   Middleware: cross-cutting HTTP concerns

Avoid blindly following folder structures; the goal is separation of
responsibilities and testability.

------------------------------------------------------------------------

## 31.2 What is dependency injection?

Instead of creating dependencies inside a function, pass them in.

Hard to test:

``` js
function UserService() {
  this.db = new Database();
}
```

Better:

``` js
function UserService(db) {
  this.db = db;
}

const service = new UserService(db);
```

Testing becomes easier:

``` js
const fakeDb = {
  findUser: async () => ({ id: 1 })
};

const service = new UserService(fakeDb);
```

------------------------------------------------------------------------

## 31.3 What is middleware?

Middleware is a function that participates in request processing.

Example:

``` js
app.use((req, res, next) => {
  console.log(req.method, req.url);
  next();
});
```

Common uses:

-   Authentication
-   Logging
-   Validation
-   Rate limiting
-   Error handling

------------------------------------------------------------------------

# 32. Performance and Scalability

## 32.1 How do you optimize JavaScript performance?

Start with measurement rather than guesses.

Common techniques:

-   Avoid unnecessary work
-   Use appropriate data structures
-   Reduce allocations in hot paths
-   Batch operations
-   Cache expensive results
-   Avoid blocking the event loop
-   Use streams for large data
-   Use workers for CPU-heavy work
-   Profile before and after optimization

------------------------------------------------------------------------

## 32.2 Why is Big-O still important in JavaScript?

Because JavaScript APIs and data structures have different complexity
characteristics.

Example:

``` js
const set = new Set(numbers);

set.has(x);
```

Average-case membership lookup is typically O(1), while:

``` js
numbers.includes(x);
```

is O(n).

For large collections and repeated lookups, choosing the right structure
matters.

------------------------------------------------------------------------

## 32.3 Array vs Set vs Map

### Array

Use when:

-   Order matters
-   Indexed access matters
-   You need duplicates

### Set

Use when:

-   Values must be unique
-   Membership checks are common

### Map

Use when:

-   You need key-value relationships
-   Keys aren't limited to strings/symbols

------------------------------------------------------------------------

# 33. Testing

## 33.1 Unit vs integration vs end-to-end tests

### Unit

Tests a small isolated unit.

``` js
expect(add(2, 3)).toBe(5);
```

### Integration

Tests multiple components working together, such as service + database.

### E2E

Tests the application through a user/system-level flow.

``` text
Browser
  ↓
API
  ↓
Database
```

SDE2 interviewers often care about knowing what should be tested at each
level.

------------------------------------------------------------------------

## 33.2 What should you mock?

Mock external dependencies when isolation is useful:

-   Payment provider
-   Email service
-   External HTTP API

Don't mock everything. Excessive mocking can make tests verify
implementation details rather than behavior.

------------------------------------------------------------------------

# 34. Common JavaScript Traps

## 34.1 `typeof []`

``` js
typeof []; // "object"
```

Use:

``` js
Array.isArray([]);
```

------------------------------------------------------------------------

## 34.2 `typeof null`

``` js
typeof null; // "object"
```

Historical quirk.

------------------------------------------------------------------------

## 34.3 `NaN`

``` js
NaN === NaN; // false
```

Use:

``` js
Number.isNaN(value);
```

------------------------------------------------------------------------

## 34.4 Empty arrays and objects are truthy

``` js
Boolean([]); // true
Boolean({}); // true
```

------------------------------------------------------------------------

## 34.5 `const` does not freeze objects

``` js
const user = { name: "Alice" };

user.name = "Bob"; // allowed
```

For shallow freezing:

``` js
Object.freeze(user);
```

But `Object.freeze` is shallow.

------------------------------------------------------------------------

# 35. Must-Know SDE1 Questions

Before an SDE1 interview, be able to explain these without notes:

1.  `var` vs `let` vs `const`
2.  Hoisting
3.  TDZ
4.  Scope
5.  Closures
6.  `this`
7.  Arrow vs regular functions
8.  `call`, `apply`, `bind`
9.  `==` vs `===`
10. Primitive vs object values
11. Shallow vs deep copy
12. Destructuring
13. Spread/rest
14. `map`, `filter`, `reduce`
15. `find`, `some`, `every`
16. Promises
17. `async/await`
18. Event loop
19. Microtask vs task queue
20. `Promise.all`
21. `Promise.allSettled`
22. `Promise.race`
23. `Promise.any`
24. Error handling
25. DOM selection
26. Event bubbling/capturing
27. Event delegation
28. `preventDefault` vs `stopPropagation`
29. ES modules
30. Basic Node.js event loop

------------------------------------------------------------------------

# 36. Must-Know SDE2 Questions

For SDE2, go beyond syntax and explain trade-offs:

1.  How does the event loop work?
2.  How can microtasks starve the event loop?
3.  How do you prevent blocking Node.js?
4.  How would you handle CPU-heavy work?
5.  How would you implement concurrency limiting?
6.  How would you implement retries?
7.  Why use exponential backoff + jitter?
8.  How would you implement request deduplication?
9.  How would you design a cache?
10. What causes cache stampedes?
11. How do streams provide backpressure?
12. How would you process a huge file safely?
13. How would you cancel asynchronous work?
14. How would you diagnose a memory leak?
15. How would you profile a slow Node.js service?
16. How would you structure a large Node.js application?
17. Where should business logic live?
18. How do you design error handling across service boundaries?
19. How do you make retries safe using idempotency?
20. How would you limit downstream API load?
21. How do you handle partial failure?
22. When would you use worker threads?
23. When would you use processes instead?
24. How do you avoid race conditions in async code?
25. How would you test asynchronous code?
26. How would you design graceful shutdown?
27. How would you implement request timeouts?
28. How would you handle backpressure in a data pipeline?
29. How do you distinguish application errors from infrastructure
    errors?
30. How do you measure before optimizing?

------------------------------------------------------------------------

# 37. High-Value Interview Coding Questions

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

Know how to explain closure + timer + cancellation.

------------------------------------------------------------------------

## Q2. Implement throttle

``` js
function throttle(fn, interval) {
  let lastTime = 0;

  return function (...args) {
    const now = Date.now();

    if (now - lastTime >= interval) {
      lastTime = now;
      fn.apply(this, args);
    }
  };
}
```

A production implementation may need leading/trailing behavior depending
on requirements.

------------------------------------------------------------------------

## Q3. Implement memoization

``` js
function memoize(fn) {
  const cache = new Map();

  return function (arg) {
    if (cache.has(arg)) {
      return cache.get(arg);
    }

    const result = fn(arg);
    cache.set(arg, result);

    return result;
  };
}
```

Follow-up: How do you support multiple arguments? One option is a stable
cache key or nested Maps, depending on the argument types and semantics.

------------------------------------------------------------------------

## Q4. Implement `Promise.all` conceptually

``` js
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let completed = 0;

    if (promises.length === 0) {
      resolve([]);
      return;
    }

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = value;
          completed++;

          if (completed === promises.length) {
            resolve(results);
          }
        })
        .catch(reject);
    });
  });
}
```

Important requirements:

-   Preserve input order
-   Resolve only after all fulfill
-   Reject on first rejection
-   Handle non-Promise values
-   Handle empty input

------------------------------------------------------------------------

## Q5. Implement a concurrency limiter

Concept:

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

Interview discussion:

-   Why not `Promise.all(tasks.map(...))`?
-   What happens if a task rejects?
-   How would you support cancellation?
-   How would you add retries?
-   How would you enforce a global rate limit?

------------------------------------------------------------------------

# 38. JavaScript Interview Mental Models

## Mental model 1: Scope

``` text
Where was the function written?
        ↓
What variables are in its lexical environment?
        ↓
Closure may retain those variables
```

## Mental model 2: `this`

``` text
How was the regular function called?
        ↓
Determine `this`

Arrow function?
        ↓
Use lexical `this`
```

## Mental model 3: Async JavaScript

``` text
Synchronous JS
      ↓
Call Stack
      ↓
Async runtime operation
      ↓
Callback becomes runnable
      ↓
Microtask/task scheduling
      ↓
Call Stack
```

## Mental model 4: Prototype lookup

``` text
object
  ↓
prototype
  ↓
prototype's prototype
  ↓
...
  ↓
null
```

## Mental model 5: Node.js scalability

``` text
Many requests
      ↓
Fast non-blocking I/O
      ↓
Event loop remains responsive
      ↓
Good concurrency

CPU-heavy JS
      ↓
Blocks event loop
      ↓
Poor latency
      ↓
Use workers/processes/other architecture
```

------------------------------------------------------------------------

# 39. Final Interview Checklist

## Basic

-   [ ] Data types
-   [ ] `null` / `undefined`
-   [ ] Truthy/falsy
-   [ ] Type coercion
-   [ ] `==` / `===`
-   [ ] `Object.is`
-   [ ] `var` / `let` / `const`
-   [ ] Hoisting
-   [ ] TDZ
-   [ ] Scope
-   [ ] Destructuring
-   [ ] Spread/rest
-   [ ] Template literals

## Functions

-   [ ] First-class functions
-   [ ] Callbacks
-   [ ] Closures
-   [ ] Higher-order functions
-   [ ] Arrow functions
-   [ ] `this`
-   [ ] `call`
-   [ ] `apply`
-   [ ] `bind`
-   [ ] Default parameters

## Objects

-   [ ] Prototype
-   [ ] Prototype chain
-   [ ] `prototype` vs `__proto__`
-   [ ] `new`
-   [ ] Classes
-   [ ] Inheritance
-   [ ] `super`
-   [ ] Static methods
-   [ ] Private fields
-   [ ] Shallow/deep copy
-   [ ] `Object.keys/values/entries`

## Async

-   [ ] Event loop
-   [ ] Call stack
-   [ ] Task queue
-   [ ] Microtask queue
-   [ ] Promise states
-   [ ] Promise chaining
-   [ ] `async/await`
-   [ ] `Promise.all`
-   [ ] `allSettled`
-   [ ] `race`
-   [ ] `any`
-   [ ] Error handling
-   [ ] Cancellation
-   [ ] Concurrency limiting

## Browser

-   [ ] DOM
-   [ ] Event propagation
-   [ ] Bubbling
-   [ ] Capturing
-   [ ] Event delegation
-   [ ] `preventDefault`
-   [ ] `stopPropagation`
-   [ ] Web APIs
-   [ ] Storage
-   [ ] CORS
-   [ ] XSS
-   [ ] CSRF

## Node.js / SDE2

-   [ ] Node event loop
-   [ ] Blocking operations
-   [ ] Worker threads
-   [ ] Streams
-   [ ] Buffers
-   [ ] Backpressure
-   [ ] EventEmitter
-   [ ] Retry/backoff
-   [ ] Timeouts
-   [ ] Idempotency
-   [ ] Caching
-   [ ] Cache stampede
-   [ ] Request deduplication
-   [ ] Memory leaks
-   [ ] Profiling
-   [ ] Graceful shutdown
-   [ ] Concurrency control
-   [ ] Testing strategy
-   [ ] Service architecture

------------------------------------------------------------------------

# 40. The 15 Topics You Should Master First

If your interview is soon, prioritize these:

1.  **Closures**
2.  **`this` + arrow functions**
3.  **Hoisting + TDZ**
4.  **Scope + lexical environment**
5.  **Prototype chain**
6.  **Promises**
7.  **`async/await`**
8.  **Event loop**
9.  **Microtask vs task queue**
10. **`Promise.all` and concurrency**
11. **Node.js event loop**
12. **Streams + backpressure**
13. **Debounce/throttle**
14. **Caching + request deduplication**
15. **Memory leaks + performance**

If you can explain these 15 deeply and solve output-based questions
involving them, you will be much better prepared for JavaScript-heavy
SDE1/SDE2 interviews than by memorizing a long list of syntax questions.
