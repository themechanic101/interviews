# React.js Interview Guide --- SDE-1 / SDE-2

A practical, interview-oriented React.js question bank from fundamentals
to advanced topics.\
Each topic follows: **short interview answer → deeper explanation → code
→ common follow-up**.

> **Target:** Freshers, SDE-1, SDE-2\
> **Style:** Modern React with functional components and Hooks\
> **Assumption:** React 18+/19 concepts where relevant; exact behavior
> can vary by React version.

------------------------------------------------------------------------

# 1. React Fundamentals

## 1. What is React?

### Interview answer

React is a JavaScript library for building user interfaces using
reusable components. It uses a declarative programming model: we
describe what the UI should look like for a given state, and React
handles updating the UI.

### Key points

-   Component-based
-   Declarative
-   Uses JSX
-   State-driven UI
-   Supports server/client rendering architectures
-   React itself is a UI library; routing and data fetching are separate
    concerns

``` jsx
function Welcome({ name }) {
  return <h1>Hello {name}</h1>;
}
```

### Follow-up: Is React a framework?

Traditionally, React is described as a library because it focuses
primarily on the UI layer. Frameworks such as Next.js build additional
application capabilities around React.

------------------------------------------------------------------------

## 2. What is a component?

A component is a reusable piece of UI with its own logic and rendering
behavior.

``` jsx
function Button({ children }) {
  return <button>{children}</button>;
}

function App() {
  return (
    <>
      <Button>Save</Button>
      <Button>Cancel</Button>
    </>
  );
}
```

A good component generally has a clear responsibility and a predictable
interface through props.

------------------------------------------------------------------------

## 3. What is JSX?

JSX is a JavaScript syntax extension that lets us describe UI using
HTML-like syntax.

``` jsx
const name = "Alex";

const element = <h1>Hello {name}</h1>;
```

JSX is transformed by the build tool/compiler into JavaScript
representation of the UI.

### JSX vs HTML

  -----------------------------------------------------------------------
  JSX                                 HTML
  ----------------------------------- -----------------------------------
  JavaScript syntax extension         Markup language

  `className`                         `class`

  `htmlFor`                           `for`

  JS expressions use `{}`             No equivalent JSX expression syntax

  Event handlers commonly use         HTML traditionally uses `onclick`
  `onClick`                           

  Components can be embedded          HTML has standard elements
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 4. What is declarative UI?

Imperative code tells the program **how** to update the UI.

Declarative React code describes **what** the UI should be for the
current state.

``` jsx
function Counter({ count }) {
  return <h1>Count: {count}</h1>;
}
```

If `count` changes, React determines the necessary UI update.

------------------------------------------------------------------------

## 5. What is an SPA?

A Single Page Application loads an application shell and then performs
most navigation and UI updates on the client without requesting a
completely new document for every route.

``` text
Initial request
     ↓
HTML + JavaScript
     ↓
React application
     ↓
Client-side navigation
     ↓
Update UI without full document reload
```

Examples include dashboards, admin panels, and many web applications.

### Trade-offs

**Pros:** smooth navigation, rich interactions, persistent client state.

**Cons:** potentially larger initial JS, client-side complexity, SEO
considerations depending on the application.

------------------------------------------------------------------------

# 2. Components, Props and State

## 6. Functional vs class components

### Functional

``` jsx
function User({ name }) {
  return <h1>{name}</h1>;
}
```

### Class

``` jsx
class User extends React.Component {
  render() {
    return <h1>{this.props.name}</h1>;
  }
}
```

### Interview answer

Modern React primarily uses functional components and Hooks. Class
components are still important when maintaining older React
applications.

------------------------------------------------------------------------

## 7. What are props?

Props are inputs passed from a parent component to a child.

``` jsx
function User({ name, age }) {
  return <p>{name} is {age}</p>;
}

<User name="Alex" age={22} />
```

Props can contain values, objects, arrays, functions, and React
elements.

### Are props mutable?

A child should treat props as read-only.

If a child needs to cause a change, the parent can pass a callback:

``` jsx
function Parent() {
  const [name, setName] = useState("Alex");

  return <Child onChange={setName} />;
}

function Child({ onChange }) {
  return <button onClick={() => onChange("Sam")}>Change</button>;
}
```

------------------------------------------------------------------------

## 8. Props vs state

  Props                                 State
  ------------------------------------- --------------------------------
  Passed into component                 Managed by component
  Read-only from child perspective      Updated through state APIs
  Controlled by parent/external owner   Owned by component/state owner
  Used to configure behavior/UI         Represents changing data

``` jsx
function Counter({ initialCount }) {
  const [count, setCount] = useState(initialCount);

  return (
    <button onClick={() => setCount(c => c + 1)}>
      {count}
    </button>
  );
}
```

`initialCount` is a prop; `count` is state.

------------------------------------------------------------------------

## 9. What is state?

State is data that can change over time and whose changes may cause a
component to render again.

``` jsx
const [isOpen, setIsOpen] = useState(false);
```

When state changes, React schedules an update.

### Important

Don't mutate state directly:

``` jsx
// Bad
user.name = "Sam";
```

Instead create a new value:

``` jsx
setUser(prev => ({
  ...prev,
  name: "Sam"
}));
```

------------------------------------------------------------------------

## 10. Why shouldn't state be mutated directly?

React relies on state updates being expressed through its update
mechanism. Direct mutation can produce stale UI, make debugging harder,
and break assumptions used by optimization techniques.

``` jsx
const [user, setUser] = useState({ name: "Alex" });

// Good
setUser(prev => ({ ...prev, name: "Sam" }));
```

------------------------------------------------------------------------

## 11. What is derived state?

Derived state is data that can be calculated from existing props/state.

Avoid storing redundant state when you can calculate it.

### Avoid

``` jsx
const [items, setItems] = useState([]);
const [count, setCount] = useState(0);
```

If `count` is always `items.length`, storing both creates two sources of
truth.

### Prefer

``` jsx
const count = items.length;
```

------------------------------------------------------------------------

## 12. What is state lifting?

If two sibling components need the same state, move the state to their
nearest common parent.

``` text
       Parent
       /    \
   Search   List
```

The parent owns the shared state:

``` jsx
function App() {
  const [query, setQuery] = useState("");

  return (
    <>
      <Search query={query} onChange={setQuery} />
      <List query={query} />
    </>
  );
}
```

------------------------------------------------------------------------

# 3. Rendering and Reconciliation

## 13. What is the Virtual DOM?

The Virtual DOM is a conceptual term for React's in-memory
representation of the UI. React uses its rendering and reconciliation
machinery to determine what changes need to be committed to the actual
DOM.

``` text
State/props change
       ↓
React render
       ↓
New UI representation
       ↓
Reconciliation
       ↓
Commit required DOM changes
```

### Important interview correction

Do not say "Virtual DOM makes React always faster than the DOM."

The performance characteristics depend on the workload, browser,
component structure, and React's rendering/commit strategy.

------------------------------------------------------------------------

## 14. What is reconciliation?

Reconciliation is the process React uses to determine how the newly
rendered element tree relates to the previous one.

React uses things such as:

-   Element type
-   Position
-   Keys
-   Component identity

to decide what can be reused and what must change.

------------------------------------------------------------------------

## 15. What causes a React component to render?

Common causes include:

1.  Its state changes.
2.  Its parent renders and passes new props/relevant identity.
3.  A context value it consumes changes.
4.  An external store it subscribes to reports an update.
5.  Its own Hook/state update schedules an update.

A render does **not** automatically mean the DOM changed.

``` text
Render ≠ DOM mutation
```

React can render, compare, and determine that no DOM update is required.

------------------------------------------------------------------------

## 16. Render phase vs commit phase

### Render phase

React calls components and calculates what the UI should look like.

### Commit phase

React applies the necessary changes to the host environment, such as the
DOM.

``` text
Trigger update
    ↓
Render phase
    ↓
Reconciliation
    ↓
Commit phase
    ↓
Browser displays result
```

Side effects should not be performed during rendering.

------------------------------------------------------------------------

## 17. What are keys?

Keys give list elements stable identity.

``` jsx
users.map(user => (
  <User key={user.id} user={user} />
));
```

React can then better determine which item corresponds to which previous
item.

### Good

``` jsx
key={user.id}
```

### Risky

``` jsx
key={index}
```

Index keys can cause incorrect state association when list items are
inserted, removed, or reordered.

### Bad

``` jsx
key={Math.random()}
```

This creates unstable identity and can cause elements to be recreated
unnecessarily.

------------------------------------------------------------------------

## 18. What happens if a key changes?

A changed key tells React that the old element and new element are
different identities.

This can cause the old component to unmount and a new one to mount.

This technique can sometimes intentionally be used to reset component
state:

``` jsx
<Chat key={conversationId} />
```

Changing `conversationId` gives `Chat` a new identity.

------------------------------------------------------------------------

# 4. Hooks

## 19. What are Hooks?

Hooks are functions that let functional components use React features
such as state, context, refs, and effects.

Common Hooks:

``` text
useState
useEffect
useContext
useRef
useMemo
useCallback
useReducer
useLayoutEffect
```

------------------------------------------------------------------------

## 20. Rules of Hooks

Hooks should:

1.  Be called at the top level.
2.  Be called from React function components or custom Hooks.

### Bad

``` jsx
if (loggedIn) {
  useEffect(() => {});
}
```

### Good

``` jsx
useEffect(() => {
  if (!loggedIn) return;
  // effect
}, [loggedIn]);
```

Why? React relies on consistent Hook call order between renders.

------------------------------------------------------------------------

## 21. Explain useState

``` jsx
const [count, setCount] = useState(0);
```

-   `count`: current state
-   `setCount`: updater
-   `0`: initial state

Use functional updates when the next state depends on the previous
state:

``` jsx
setCount(c => c + 1);
setCount(c => c + 1);
```

This is safer than repeatedly reading a potentially stale `count` value.

------------------------------------------------------------------------

## 22. Why does React batch state updates?

React can group multiple state updates so that they are processed
together, reducing unnecessary rendering work.

``` jsx
function handleClick() {
  setFirstName("Alex");
  setLastName("Smith");
}
```

The exact batching behavior depends on the React version and update
context, but modern React batches many updates automatically.

------------------------------------------------------------------------

## 23. Why is state sometimes "stale"?

State values inside a render represent that render's snapshot.

``` jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    console.log(count);
  }

  return <button onClick={handleClick}>{count}</button>;
}
```

The log may show the previous value because `setCount` schedules an
update; it does not mutate the `count` variable of the current render.

Use a functional updater when necessary:

``` jsx
setCount(c => c + 1);
```

------------------------------------------------------------------------

## 24. Explain useEffect

`useEffect` lets a component synchronize with external systems.

Examples:

-   API/network synchronization
-   Event listeners
-   Timers
-   Subscriptions
-   Browser APIs

``` jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log("tick");
  }, 1000);

  return () => clearInterval(id);
}, []);
```

The returned function is cleanup.

### Important

`useEffect` is not simply "a lifecycle method." Its modern mental model
is synchronization with external systems.

------------------------------------------------------------------------

## 25. Explain dependency arrays

``` jsx
useEffect(() => {
  // ...
}, [userId]);
```

The effect is synchronized when `userId` changes.

``` jsx
useEffect(() => {
  // ...
}, []);
```

The dependency list is empty.

``` jsx
useEffect(() => {
  // ...
});
```

No dependency array means the effect runs after every committed render.

### Interview warning

Don't blindly omit dependencies to "make the effect run once." The
dependency list should represent the values used by the effect that need
synchronization.

------------------------------------------------------------------------

## 26. Why does useEffect run twice in development?

With Strict Mode, React may perform an additional setup/cleanup cycle in
development to expose effects that are not resilient to being started
and stopped.

``` jsx
useEffect(() => {
  console.log("setup");

  return () => {
    console.log("cleanup");
  };
}, []);
```

This is a development diagnostic behavior, not a reason to add arbitrary
flags to suppress effects.

------------------------------------------------------------------------

## 27. useEffect cleanup

Cleanup is needed for resources that must be released.

``` jsx
useEffect(() => {
  function handleResize() {
    console.log(window.innerWidth);
  }

  window.addEventListener("resize", handleResize);

  return () => {
    window.removeEventListener("resize", handleResize);
  };
}, []);
```

Other examples:

-   `clearInterval`
-   unsubscribe
-   abort requests where appropriate
-   disconnect observers

------------------------------------------------------------------------

## 28. useRef

`useRef` stores a mutable value that persists across renders without
causing a render when changed.

### DOM reference

``` jsx
function Input() {
  const inputRef = useRef(null);

  return (
    <>
      <input ref={inputRef} />
      <button onClick={() => inputRef.current?.focus()}>
        Focus
      </button>
    </>
  );
}
```

### Mutable value

``` jsx
const renderCount = useRef(0);

renderCount.current++;
```

Changing `current` does not itself trigger a re-render.

------------------------------------------------------------------------

## 29. useMemo

`useMemo` memoizes a calculated value.

``` jsx
const filteredUsers = useMemo(() => {
  return users.filter(user => user.name.includes(query));
}, [users, query]);
```

Use it when:

-   Calculation is meaningfully expensive, or
-   Stable value identity helps avoid unnecessary work.

Do not use `useMemo` everywhere.

------------------------------------------------------------------------

## 30. useCallback

`useCallback` memoizes a function identity.

``` jsx
const handleDelete = useCallback((id) => {
  deleteUser(id);
}, []);
```

It can be useful when passing callbacks to memoized children or when a
stable function identity is required.

### useMemo vs useCallback

``` text
useMemo     → memoizes a value
useCallback → memoizes a function
```

Conceptually:

``` jsx
useCallback(fn, deps)
```

is similar to:

``` jsx
useMemo(() => fn, deps)
```

------------------------------------------------------------------------

## 31. React.memo

`React.memo` can skip re-rendering a component when its props are
considered equal.

``` jsx
const User = React.memo(function User({ name }) {
  return <h1>{name}</h1>;
});
```

This is useful when a component renders often with the same props and
rendering it again is expensive.

### Important

`React.memo` is an optimization, not a correctness mechanism.

It does not prevent re-renders caused by the component's own state or
consumed context changes.

------------------------------------------------------------------------

## 32. useReducer

Useful when state transitions are complex or multiple actions update
related state.

``` jsx
function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { ...state, count: state.count + 1 };

    case "reset":
      return { ...state, count: 0 };

    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <button onClick={() => dispatch({ type: "increment" })}>
      {state.count}
    </button>
  );
}
```

### useState vs useReducer

Use `useState` for simple local state.

Use `useReducer` when state transitions are complex, action-driven, or
easier to express as a state machine.

------------------------------------------------------------------------

## 33. What is a custom Hook?

A custom Hook is a function that uses Hooks to reuse stateful logic.

``` jsx
function useOnlineStatus() {
  const [online, setOnline] = useState(navigator.onLine);

  useEffect(() => {
    const onlineHandler = () => setOnline(true);
    const offlineHandler = () => setOnline(false);

    window.addEventListener("online", onlineHandler);
    window.addEventListener("offline", offlineHandler);

    return () => {
      window.removeEventListener("online", onlineHandler);
      window.removeEventListener("offline", offlineHandler);
    };
  }, []);

  return online;
}
```

Usage:

``` jsx
function Status() {
  const online = useOnlineStatus();
  return <p>{online ? "Online" : "Offline"}</p>;
}
```

Custom Hooks reuse **logic**, not UI.

------------------------------------------------------------------------

# 5. Forms and Events

## 34. Controlled vs uncontrolled components

### Controlled

React state is the source of truth.

``` jsx
function Form() {
  const [email, setEmail] = useState("");

  return (
    <input
      value={email}
      onChange={e => setEmail(e.target.value)}
    />
  );
}
```

### Uncontrolled

The DOM holds the current value.

``` jsx
function Form() {
  const inputRef = useRef(null);

  function submit() {
    console.log(inputRef.current.value);
  }

  return <input ref={inputRef} />;
}
```

Controlled inputs are often preferable for dynamic validation and UI
logic.

------------------------------------------------------------------------

## 35. What are synthetic events?

React provides a consistent event interface around browser events.

``` jsx
function Button() {
  function handleClick(event) {
    console.log(event.type);
  }

  return <button onClick={handleClick}>Click</button>;
}
```

Modern React no longer uses the old event pooling behavior that existed
in earlier React versions, but the term "SyntheticEvent" remains part of
React's event system.

------------------------------------------------------------------------

## 36. Event bubbling

Events can propagate from the target toward ancestors.

``` jsx
<div onClick={() => console.log("parent")}>
  <button onClick={() => console.log("button")}>
    Click
  </button>
</div>
```

Clicking the button can result in:

``` text
button
parent
```

Stop propagation when appropriate:

``` jsx
event.stopPropagation();
```

------------------------------------------------------------------------

## 37. Why use onClick instead of onclick?

React JSX uses camelCase event props:

``` jsx
<button onClick={handleClick}>
```

not:

``` jsx
<button onclick={handleClick}>
```

This is part of React's event API.

------------------------------------------------------------------------

# 6. Context and Component Architecture

## 38. What is Context API?

Context lets components consume a value without passing it through every
intermediate component.

``` jsx
const ThemeContext = createContext(null);

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Dashboard />
    </ThemeContext.Provider>
  );
}

function Button() {
  const theme = useContext(ThemeContext);
  return <button data-theme={theme}>Save</button>;
}
```

Common uses:

-   Theme
-   Locale
-   Authentication/session information
-   Application configuration

### Important

Context is not automatically a replacement for all global state
management.

------------------------------------------------------------------------

## 39. What is props drilling?

Props drilling means passing data through components that do not need it
simply to reach a deeply nested component.

``` text
App
 ↓ props
Dashboard
 ↓ props
Profile
 ↓ props
Button
```

Solutions include:

-   Context
-   Composition
-   Moving state
-   External state stores when appropriate

------------------------------------------------------------------------

## 40. Context vs Redux/external state management

Context primarily provides a way to make values available to
descendants.

An external state library may additionally provide:

-   Centralized state
-   Selective subscriptions
-   Structured actions
-   Middleware
-   Devtools
-   Complex state management patterns

Do not choose a state library just because state is "global." Evaluate
update frequency, ownership, complexity, and team needs.

------------------------------------------------------------------------

# 7. Performance

## 41. How do you optimize React performance?

A good interview answer:

> First I measure the bottleneck. Then I optimize component boundaries,
> avoid unnecessary state, use stable keys, memoize expensive
> calculations only when useful, virtualize large lists, split bundles,
> and reduce unnecessary network work.

Common techniques:

``` text
1. Keep state close to where it is used
2. Avoid unnecessary derived state
3. React.memo where justified
4. useMemo/useCallback where justified
5. Code splitting
6. Lazy loading
7. List virtualization
8. Avoid unnecessary context updates
9. Optimize API/data fetching
10. Profile before optimizing
```

------------------------------------------------------------------------

## 42. What is code splitting?

Code splitting breaks JavaScript into smaller chunks loaded when needed.

``` jsx
const Settings = lazy(() => import("./Settings"));

function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <Settings />
    </Suspense>
  );
}
```

Instead of downloading every feature immediately, the application can
load some code on demand.

------------------------------------------------------------------------

## 43. What is lazy loading?

Lazy loading means delaying loading of something until it is needed.

For React components:

``` jsx
const AdminPage = lazy(() => import("./AdminPage"));
```

Usually combined with:

``` jsx
<Suspense fallback={<Spinner />}>
  <AdminPage />
</Suspense>
```

------------------------------------------------------------------------

## 44. What is Suspense?

`Suspense` lets React show a fallback while a child is waiting on a
Suspense-enabled operation.

``` jsx
<Suspense fallback={<Loading />}>
  <Profile />
</Suspense>
```

A common use is lazy-loaded components.

Modern React frameworks can also use Suspense for server/data-loading
architectures.

------------------------------------------------------------------------

## 45. How would you optimize a large list?

Problem:

``` jsx
items.map(item => <Row key={item.id} item={item} />);
```

Rendering 100,000 rows can be expensive.

A common solution is **virtualization/windowing**:

``` text
100,000 items
      ↓
Only visible ~20–50 rows
      ↓
Render those rows
      ↓
Change rendered window during scroll
```

Libraries such as `react-window` or other virtualization solutions can
help.

------------------------------------------------------------------------

## 46. Why is index as a key problematic?

Suppose:

``` text
A
B
C
```

Keys:

``` text
0
1
2
```

Insert X at the beginning:

``` text
X
A
B
C
```

Keys become:

``` text
0
1
2
3
```

React may associate the previous component at key `0` with X instead of
A. If rows contain local state, that state can appear to move to the
wrong item.

Stable IDs avoid this.

------------------------------------------------------------------------

# 8. Advanced Component Patterns

## 47. What is a Higher-Order Component?

A Higher-Order Component is a function that takes a component and
returns another component.

``` jsx
function withAuth(Component) {
  return function Protected(props) {
    const loggedIn = true;

    if (!loggedIn) {
      return <p>Please log in</p>;
    }

    return <Component {...props} />;
  };
}

const ProtectedDashboard = withAuth(Dashboard);
```

HOCs were widely used historically for reusable logic.

Modern React often prefers:

-   Hooks
-   Composition
-   Render props in some cases

------------------------------------------------------------------------

## 48. What are render props?

A render prop is a prop whose value is a function used to decide what UI
to render.

``` jsx
function MouseTracker({ render }) {
  const [x, setX] = useState(0);

  return (
    <div onMouseMove={e => setX(e.clientX)}>
      {render(x)}
    </div>
  );
}

<MouseTracker
  render={x => <p>Mouse X: {x}</p>}
/>
```

This pattern was popular before Hooks.

------------------------------------------------------------------------

## 49. Composition vs inheritance

React generally favors composition over inheritance for UI reuse.

``` jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

<Card>
  <UserProfile />
</Card>
```

Composition allows a parent component to define structure while children
provide content.

------------------------------------------------------------------------

## 50. What are React Portals?

Portals render React children into a different DOM node.

``` jsx
createPortal(
  <Modal />,
  document.getElementById("modal-root")
);
```

Useful for:

-   Modals
-   Dialogs
-   Tooltips
-   Overlays

Even though the DOM node is elsewhere, the portal remains connected to
the same React tree for context and React event propagation.

------------------------------------------------------------------------

# 9. Routing

## 51. How does React Router work?

A router maps URLs to UI.

``` jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/products" element={<Products />} />
  <Route path="/cart" element={<Cart />} />
</Routes>
```

Client-side navigation can change the URL and render a different
component without a full document reload.

------------------------------------------------------------------------

## 52. Link vs anchor tag

React Router:

``` jsx
<Link to="/products">Products</Link>
```

Traditional anchor:

``` html
<a href="/products">Products</a>
```

For internal SPA navigation, router links can perform client-side
navigation.

An anchor is still appropriate when you intentionally want normal
browser navigation or an external URL.

------------------------------------------------------------------------

## 53. What are nested routes?

Nested routes let a parent route render shared layout/UI while a child
route renders inside it.

Conceptually:

``` text
/dashboard
   ├── overview
   ├── users
   └── settings
```

This avoids duplicating the dashboard shell.

------------------------------------------------------------------------

## 54. What are protected routes?

A protected route checks whether a user is allowed to access a page.

Conceptually:

``` jsx
function ProtectedRoute({ children }) {
  const user = useUser();

  if (!user) {
    return <Navigate to="/login" replace />;
  }

  return children;
}
```

Authorization should also be enforced on the backend. Client-side route
protection is primarily a UX/navigation mechanism, not a security
boundary.

------------------------------------------------------------------------

# 10. Lifecycle and Strict Mode

## 55. What are React lifecycle phases?

Conceptually:

``` text
Mount
  ↓
Update
  ↓
Unmount
```

Class components expose lifecycle methods such as:

``` jsx
componentDidMount()
componentDidUpdate()
componentWillUnmount()
```

Functional components use Hooks and React's render/effect model rather
than class lifecycle methods.

------------------------------------------------------------------------

## 56. What is Strict Mode?

Strict Mode is a development-only diagnostic feature.

``` jsx
<StrictMode>
  <App />
</StrictMode>
```

It helps expose:

-   Impure rendering
-   Missing cleanup
-   Deprecated/unsafe patterns
-   Effects that are not resilient to setup/cleanup

It does not create a visible DOM element.

------------------------------------------------------------------------

# 11. Data Fetching

## 57. How do you fetch API data in React?

A simple example:

``` jsx
function Users() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    async function load() {
      try {
        const response = await fetch("/api/users");

        if (!response.ok) {
          throw new Error("Request failed");
        }

        const data = await response.json();

        if (!cancelled) {
          setUsers(data);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }

    load();

    return () => {
      cancelled = true;
    };
  }, []);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error</p>;

  return users.map(user => (
    <p key={user.id}>{user.name}</p>
  ));
}
```

In production applications, a dedicated server-state/data-fetching
solution can often handle caching, retries, deduplication, invalidation,
and loading/error state more effectively.

------------------------------------------------------------------------

## 58. What is the difference between client state and server state?

### Client state

State primarily owned by the UI:

``` text
modalOpen
selectedTab
inputValue
sidebarCollapsed
```

### Server state

Data owned by a backend:

``` text
users
products
orders
comments
```

Server state has special concerns:

-   Caching
-   Staleness
-   Refetching
-   Synchronization
-   Pagination
-   Deduplication

This distinction is important when choosing state management
architecture.

------------------------------------------------------------------------

## 59. How do you prevent race conditions in API requests?

Use cancellation or ignore stale responses.

Using `AbortController`:

``` jsx
useEffect(() => {
  const controller = new AbortController();

  async function load() {
    const response = await fetch(
      `/api/users/${userId}`,
      { signal: controller.signal }
    );

    const data = await response.json();
    setUser(data);
  }

  load().catch(error => {
    if (error.name !== "AbortError") {
      console.error(error);
    }
  });

  return () => controller.abort();
}, [userId]);
```

When `userId` changes, the previous request can be aborted.

------------------------------------------------------------------------

# 12. State Management

## 60. When should state be local vs global?

Keep state local when only one component/subtree needs it.

Use shared/global state when multiple distant parts of the application
need the same state and there is a clear shared owner.

A useful rule:

``` text
Keep state as close as possible
to the components that use it.
```

Do not make every piece of state global.

------------------------------------------------------------------------

## 61. What is Redux?

Redux is a predictable state management library based around a
centralized store and explicit state transitions.

Typical flow:

``` text
Component
   ↓ dispatch(action)
Store/reducer
   ↓
New state
   ↓
Subscribed UI
```

Modern Redux applications commonly use Redux Toolkit rather than writing
Redux boilerplate manually.

------------------------------------------------------------------------

## 62. Context vs Redux

Context:

``` text
Dependency/value distribution
```

Redux:

``` text
Structured application state management
```

Context can be enough for themes or authentication information.

A complex application with many independent state transitions may
benefit from a dedicated state-management solution.

------------------------------------------------------------------------

# 13. Advanced Rendering Concepts

## 63. What is concurrent rendering?

Modern React can interrupt, prioritize, and schedule rendering work
rather than treating every update as equally urgent.

The key idea is that rendering work can be scheduled according to
priority.

This enables features such as:

-   Transitions
-   Deferred updates
-   Responsive UI during expensive updates

------------------------------------------------------------------------

## 64. What is startTransition?

`startTransition` marks updates as non-urgent.

``` jsx
const [query, setQuery] = useState("");
const [results, setResults] = useState([]);

function handleChange(e) {
  const value = e.target.value;

  setQuery(value);

  startTransition(() => {
    setResults(expensiveFilter(value));
  });
}
```

The input update remains urgent while the expensive result update can be
treated as lower priority.

### Interview answer

> A transition lets React distinguish urgent UI updates from non-urgent
> rendering work so the interface can remain responsive.

------------------------------------------------------------------------

## 65. What is useTransition?

`useTransition` provides transition state plus a function for starting a
transition.

``` jsx
const [isPending, startTransition] = useTransition();

function changeTab(tab) {
  startTransition(() => {
    setTab(tab);
  });
}
```

Now:

``` jsx
{isPending && <p>Loading...</p>}
```

can communicate that the transition is still pending.

------------------------------------------------------------------------

## 66. What is useDeferredValue?

It lets a value have a deferred version that can lag behind during
expensive rendering.

``` jsx
const [query, setQuery] = useState("");
const deferredQuery = useDeferredValue(query);

const results = useMemo(
  () => searchItems(deferredQuery),
  [deferredQuery]
);
```

Useful when displaying expensive results while keeping an input
responsive.

------------------------------------------------------------------------

# 14. Error Handling

## 67. What are Error Boundaries?

Error boundaries catch rendering errors in descendant components and
show fallback UI.

Historically implemented using class components:

``` jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    console.error(error, info);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```

Important: Error boundaries do not catch every kind of error, such as
errors from ordinary event handlers or arbitrary asynchronous callbacks.

------------------------------------------------------------------------

# 15. Security

## 68. What is XSS?

Cross-Site Scripting occurs when untrusted content is interpreted as
executable markup/script in a user's browser.

React escapes text content by default:

``` jsx
<div>{userInput}</div>
```

is safer than manually injecting raw HTML.

Be careful with:

``` jsx
<div dangerouslySetInnerHTML={{ __html: html }} />
```

Only use raw HTML when the content is trusted or properly sanitized.

------------------------------------------------------------------------

## 69. Is React automatically secure?

No.

React provides useful defaults such as escaping rendered text, but
application security is broader:

-   Validate input
-   Sanitize HTML when necessary
-   Protect authentication
-   Use secure cookies
-   Enforce authorization on the server
-   Protect APIs
-   Avoid leaking secrets into client bundles

Never put backend secrets/API private keys in frontend code.

------------------------------------------------------------------------

# 16. Server Rendering and Modern React

## 70. CSR vs SSR

### CSR --- Client-Side Rendering

Browser downloads JS and React builds the UI primarily on the client.

``` text
Browser
 ↓
HTML + JS
 ↓
React executes
 ↓
UI
```

### SSR --- Server-Side Rendering

Server generates HTML for the request.

``` text
Browser
 ↓
Server
 ↓
HTML
 ↓
Browser displays
 ↓
React attaches/continues client behavior
```

SSR can improve initial content delivery and SEO, but it adds
architectural complexity.

------------------------------------------------------------------------

## 71. What is hydration?

Hydration is the process where React takes server-rendered HTML and
attaches/continues React behavior on the client.

Conceptually:

``` text
Server
  ↓
HTML
  ↓
Browser
  ↓
React hydration
  ↓
Interactive application
```

The client and server need to produce compatible output.

------------------------------------------------------------------------

## 72. What causes hydration mismatch?

A hydration mismatch occurs when the server-rendered output doesn't
match what the client expects to render.

Common causes:

``` jsx
// Risky during render
<div>{new Date().toLocaleTimeString()}</div>
```

The server and browser can produce different values.

Other causes:

-   Random values during render
-   Browser-only APIs during server render
-   Different data on server/client
-   Conditional rendering based on client-only state

------------------------------------------------------------------------

# 17. React Architecture / SDE-2 Questions

## 73. How would you structure a large React application?

A possible structure:

``` text
src/
├── app/
├── components/
├── features/
│   ├── auth/
│   ├── users/
│   └── orders/
├── hooks/
├── services/
├── routes/
├── utils/
└── types/
```

The exact structure depends on the application.

A feature-oriented structure often scales better than putting every
component in one giant `components/` directory.

------------------------------------------------------------------------

## 74. How do you design reusable components?

Think about:

``` text
API
 ↓
Props
 ↓
Composition
 ↓
State ownership
 ↓
Accessibility
 ↓
Styling
```

Example:

``` jsx
function Modal({ open, onClose, title, children }) {
  if (!open) return null;

  return (
    <div role="dialog" aria-modal="true">
      <h2>{title}</h2>
      {children}
      <button onClick={onClose}>Close</button>
    </div>
  );
}
```

The component is reusable because it doesn't own application-specific
data.

------------------------------------------------------------------------

## 75. How do you avoid prop explosion?

Instead of:

``` jsx
<Button
  text="Save"
  color="blue"
  size="large"
  loading={true}
  disabled={false}
  icon="save"
  ...
/>
```

consider:

-   Composition
-   Sensible defaults
-   Variant props
-   Smaller focused components
-   Configuration objects when appropriate

Example:

``` jsx
<Button variant="primary" loading>
  Save
</Button>
```

------------------------------------------------------------------------

## 76. How do you decide where state should live?

Ask:

1.  Who owns this data?
2.  Who needs to read it?
3.  Who needs to update it?
4.  How frequently does it change?
5.  Is it server state or client state?
6.  Can it be derived instead?

Then choose:

``` text
Local state
   ↓
Lift state
   ↓
Context
   ↓
External store/server-state solution
```

Don't jump directly to global state.

------------------------------------------------------------------------

# 18. Testing

## 77. How do you test React components?

Focus on behavior rather than implementation details.

Example:

``` jsx
render(<Login />);

await user.type(
  screen.getByLabelText(/email/i),
  "test@example.com"
);

await user.click(
  screen.getByRole("button", { name: /login/i })
);

expect(screen.getByText(/welcome/i)).toBeInTheDocument();
```

Common tools include:

-   React Testing Library
-   Vitest/Jest
-   Browser E2E tools such as Playwright/Cypress

### Good principle

Test:

``` text
What the user sees and does
```

rather than:

``` text
Internal component implementation
```

------------------------------------------------------------------------

# 19. Common Tricky Interview Questions

## 78. Why does changing state cause a render?

Because state is part of the component's reactive input. Updating it
schedules React to render the component so the UI can be recalculated.

------------------------------------------------------------------------

## 79. Does every render update the DOM?

No.

``` text
State update
 ↓
Render
 ↓
Compare/reconcile
 ↓
Maybe DOM update
```

A render can result in no DOM changes.

------------------------------------------------------------------------

## 80. Does React re-render the whole application?

Not necessarily.

React renders components according to the update and component tree,
then determines what needs to be committed. A parent render can cause
child components to render depending on the tree and optimization
boundaries, but that does not mean the entire DOM is recreated.

------------------------------------------------------------------------

## 81. Why shouldn't you use array index as key?

Because list position is not stable when items are inserted, deleted, or
reordered. Stable identity should normally come from the data itself.

------------------------------------------------------------------------

## 82. Why shouldn't you call API directly during render?

Rendering should be pure.

Bad:

``` jsx
function Users() {
  fetch("/api/users");
  return <div>Users</div>;
}
```

It can cause requests on every render.

Use an appropriate effect/data-fetching architecture instead.

------------------------------------------------------------------------

## 83. Why shouldn't you use useEffect for everything?

Because effects are intended for synchronization with external systems.

If something can be calculated directly during rendering:

``` jsx
const fullName = firstName + " " + lastName;
```

don't create an effect just to calculate it:

``` jsx
// unnecessary
useEffect(() => {
  setFullName(firstName + " " + lastName);
}, [firstName, lastName]);
```

------------------------------------------------------------------------

## 84. Why is this effect potentially wrong?

``` jsx
useEffect(() => {
  setCount(count + 1);
}, [count]);
```

Every update to `count` triggers the effect, which updates `count`
again, potentially causing an infinite loop.

The correct design depends on the requirement. Often the derived value
should simply be calculated rather than stored.

------------------------------------------------------------------------

# 20. Coding Interview Questions

## 85. Build a counter

``` jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(c => c - 1)}>
        -
      </button>

      <span>{count}</span>

      <button onClick={() => setCount(c => c + 1)}>
        +
      </button>
    </div>
  );
}
```

------------------------------------------------------------------------

## 86. Build a search/filter component

``` jsx
function SearchUsers({ users }) {
  const [query, setQuery] = useState("");

  const filteredUsers = users.filter(user =>
    user.name.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search"
      />

      {filteredUsers.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
    </>
  );
}
```

------------------------------------------------------------------------

## 87. Build a debounced search

``` jsx
function Search({ onSearch }) {
  const [query, setQuery] = useState("");

  useEffect(() => {
    const timer = setTimeout(() => {
      if (query.trim()) {
        onSearch(query);
      }
    }, 500);

    return () => clearTimeout(timer);
  }, [query, onSearch]);

  return (
    <input
      value={query}
      onChange={e => setQuery(e.target.value)}
    />
  );
}
```

Interview discussion:

-   Why cleanup?
-   What happens during fast typing?
-   How would you cancel network requests?
-   How would you cache results?
-   What if the response arrives out of order?

------------------------------------------------------------------------

## 88. Build a reusable modal

``` jsx
function Modal({ open, onClose, children }) {
  if (!open) return null;

  return createPortal(
    <div role="dialog" aria-modal="true">
      <div>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>,
    document.getElementById("modal-root")
  );
}
```

SDE-2 follow-ups:

-   Focus trapping
-   Escape key
-   Scroll locking
-   Accessibility
-   Portal
-   Cleanup
-   Animation
-   Stacking/z-index

------------------------------------------------------------------------

## 89. Build a todo list

``` jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [text, setText] = useState("");

  function addTodo() {
    const value = text.trim();

    if (!value) return;

    setTodos(prev => [
      ...prev,
      {
        id: crypto.randomUUID(),
        text: value,
        completed: false
      }
    ]);

    setText("");
  }

  function toggleTodo(id) {
    setTodos(prev =>
      prev.map(todo =>
        todo.id === id
          ? { ...todo, completed: !todo.completed }
          : todo
      )
    );
  }

  return (
    <>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />

      <button onClick={addTodo}>Add</button>

      {todos.map(todo => (
        <label key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => toggleTodo(todo.id)}
          />
          {todo.text}
        </label>
      ))}
    </>
  );
}
```

------------------------------------------------------------------------

# 21. SDE-2 Performance Scenario

## 90. A React page is slow. How would you debug it?

A strong answer:

> I wouldn't immediately add memoization. First I'd reproduce and
> measure the issue using browser performance tools and React DevTools
> Profiler. Then I'd determine whether the bottleneck is rendering,
> JavaScript computation, network/data fetching, bundle size, or
> DOM/layout work.

Then investigate:

``` text
Slow?
 ↓
Measure
 ↓
Identify bottleneck
 ├── Rendering → component boundaries/memoization
 ├── Expensive computation → memoization/algorithm
 ├── Huge list → virtualization
 ├── Large bundle → code splitting
 ├── Network → caching/deduplication/pagination
 └── Too many updates → state ownership/context architecture
```

This is much stronger than simply saying "use `useMemo`."

------------------------------------------------------------------------

# 22. SDE-2 System Design Questions

## 91. Design a large React dashboard

Discuss:

### Component architecture

``` text
App
 ├── Layout
 │    ├── Sidebar
 │    └── Header
 │
 └── Dashboard
      ├── KPI cards
      ├── Charts
      ├── Tables
      └── Filters
```

### State

Separate:

``` text
UI state
Server state
URL state
Form state
```

### Performance

-   Lazy-load heavy charts
-   Virtualize large tables
-   Memoize expensive computations only after measurement
-   Cache API results
-   Paginate
-   Split bundles

### Reliability

-   Error boundaries
-   Loading states
-   Empty states
-   Retry behavior
-   Offline/error handling

------------------------------------------------------------------------

# 23. React Architecture Questions You Should Practice

## 92. Where should API calls live?

Avoid scattering business logic throughout presentational components.

Possible architecture:

``` text
Component
   ↓
Hook
   ↓
Service/data layer
   ↓
API
```

Example:

``` jsx
function useUsers() {
  return useQuery({
    queryKey: ["users"],
    queryFn: fetchUsers
  });
}
```

The exact data layer depends on the application's stack.

------------------------------------------------------------------------

## 93. How do you prevent unnecessary re-renders from Context?

Problem:

``` jsx
<Context.Provider value={{ user, setUser }}>
```

The object may get a new identity whenever the provider renders.

Potentially:

``` jsx
const value = useMemo(
  () => ({ user, setUser }),
  [user]
);
```

But this is not a universal fix. More important strategies include:

-   Split contexts by concern
-   Keep providers appropriately scoped
-   Avoid putting high-frequency state into broad contexts
-   Use selector-based/external stores where appropriate
-   Measure before optimizing

------------------------------------------------------------------------

## 94. How would you optimize a React table with 50,000 rows?

Answer:

1.  Server-side pagination/filtering when appropriate.
2.  Virtualize rows.
3.  Avoid rendering hidden columns.
4.  Stable keys.
5.  Memoize expensive cells only where useful.
6.  Keep state local to cells/rows when possible.
7.  Avoid recreating expensive data structures unnecessarily.
8.  Profile before and after optimization.

------------------------------------------------------------------------

# 24. React Mental Model

The most important mental model:

``` text
             Props
               │
               ▼
           Component
               │
               ▼
             Render
               │
               ▼
        React representation
               │
               ▼
         Reconciliation
               │
               ▼
            Commit
               │
               ▼
             DOM
```

State:

``` text
State update
     ↓
Schedule update
     ↓
Render
     ↓
Reconcile
     ↓
Commit
```

Effects:

``` text
Commit
  ↓
Effect setup
  ↓
External system
  ↓
Cleanup when synchronization changes/unmounts
```

------------------------------------------------------------------------

# 25. Top 30 Questions to Memorize

For a fresher/SDE-1 interview, be extremely comfortable with:

1.  What is React?
2.  Why React?
3.  What is JSX?
4.  What is a component?
5.  Props vs state
6.  Controlled vs uncontrolled components
7.  What is state lifting?
8.  What is props drilling?
9.  What are keys?
10. Why avoid index keys?
11. Virtual DOM
12. Reconciliation
13. Render vs commit
14. `useState`
15. Functional state updates
16. `useEffect`
17. Dependency array
18. Effect cleanup
19. `useRef`
20. `useMemo`
21. `useCallback`
22. `React.memo`
23. Context API
24. Custom Hooks
25. React Router
26. Strict Mode
27. Portals
28. Error boundaries
29. Code splitting/lazy loading
30. React performance optimization

------------------------------------------------------------------------

# 26. SDE-2 Deep-Dive Topics

For SDE-2 interviews, go beyond definitions and prepare:

1.  React rendering model
2.  Reconciliation and identity
3.  Keys and state preservation
4.  Render vs commit
5.  Strict Mode
6.  Concurrent rendering
7.  Transitions
8.  Suspense
9.  Server rendering/hydration
10. Server Components concepts
11. State architecture
12. Server state vs client state
13. Context performance
14. Large-list virtualization
15. Bundle optimization
16. Error handling architecture
17. Accessibility
18. Testing strategy
19. Design systems
20. React application system design

------------------------------------------------------------------------

# 27. Interview Answer Formula

For most React questions, use:

``` text
1. Definition
2. Why it exists
3. Small example
4. Trade-off / caveat
```

Example:

**Interviewer:** What is `useMemo`?

**Good answer:**

> `useMemo` memoizes the result of a calculation between renders. I use
> it when a calculation is expensive or when stable value identity helps
> an optimization. For example, I can memoize an expensive
> filtered/sorted list based on its dependencies. I wouldn't use it
> everywhere because memoization itself has overhead and can make code
> harder to reason about.

This style demonstrates understanding rather than memorization.

------------------------------------------------------------------------

# 28. Rapid-Fire Revision

### React

> UI library for building component-based applications.

### JSX

> JavaScript syntax for describing UI.

### Props

> Read-only inputs passed to components.

### State

> Component/application data that changes over time.

### Key

> Stable identity for list elements.

### Virtual DOM

> In-memory UI representation used as part of React's rendering process.

### Reconciliation

> Determining how the new rendered tree relates to the previous tree.

### `useState`

> Hook for state.

### `useEffect`

> Hook for synchronizing with external systems.

### `useRef`

> Persistent mutable reference that doesn't itself trigger rendering.

### `useMemo`

> Memoizes a calculated value.

### `useCallback`

> Memoizes a function identity.

### `React.memo`

> Skips rendering when props are unchanged according to its comparison.

### Context

> Makes values available to descendants without prop drilling.

### HOC

> Function that takes a component and returns an enhanced component.

### Portal

> Renders React content into another DOM node.

### Strict Mode

> Development-time diagnostic feature.

### Suspense

> Coordinates fallback UI while Suspense-enabled content is waiting.

### Hydration

> Attaching React behavior to server-rendered HTML.

### Code splitting

> Loading JavaScript in smaller chunks, often on demand.

------------------------------------------------------------------------

# 29. Final Interview Checklist

Before an SDE-1 interview, you should be able to code without looking
up:

``` text
✓ Counter
✓ Todo list
✓ Search/filter
✓ Debounced search
✓ Form validation
✓ Fetch API data
✓ Loading/error/empty states
✓ Modal using Portal
✓ Custom Hook
✓ Pagination
✓ Infinite scroll
✓ Tabs
✓ Accordion
✓ Dropdown
✓ Protected route
```

For SDE-2, additionally practice designing:

``` text
✓ Large dashboard
✓ Data-heavy table
✓ Search/autocomplete
✓ Notification system
✓ Real-time UI
✓ Role/permission UI
✓ Design system/component library
✓ Large-scale React application architecture
```

------------------------------------------------------------------------

# 30. Golden Rules

1.  **Don't mutate state.**
2.  **Use functional updates when the next state depends on previous
    state.**
3.  **Use stable keys.**
4.  **Don't use index as key for dynamic/reorderable lists.**
5.  **Don't put side effects in render.**
6.  **Don't use `useEffect` for ordinary derived calculations.**
7.  **Clean up subscriptions, timers, and listeners.**
8.  **Don't blindly use `useMemo`/`useCallback`.**
9.  **Keep state close to where it is used.**
10. **Distinguish server state from client state.**
11. **Measure performance before optimizing.**
12. **Client-side route protection is not backend authorization.**
13. **Don't trust raw HTML from users.**
14. **Design components around clear responsibilities.**
15. **For SDE-2, explain trade-offs, not just APIs.**

------------------------------------------------------------------------

# Quick Mental Model

When debugging a React application, ask:

``` text
What changed?
    ↓
State?
Props?
Context?
External store?
    ↓
Why did it render?
    ↓
Did the DOM actually change?
    ↓
Did an effect run?
    ↓
Is the effect synchronized correctly?
    ↓
Is the bottleneck:
Rendering / JS / Network / DOM / Bundle?
```

That mental model is more valuable in an interview than memorizing
individual Hook definitions.
