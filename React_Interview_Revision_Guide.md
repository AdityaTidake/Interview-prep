# React — Fresher Software Engineering Interview Revision Guide

> **Goal:** Be able to confidently defend React as a technology listed on your resume, from fundamentals through internal behavior, practical usage, performance, and common interview traps.

---

## 1. Fundamentals

### Q1. What is React?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Whether you understand React beyond calling it a UI library.

**Expected Answer:**

React is a JavaScript library for building user interfaces, especially component-based web applications. It lets us divide a UI into reusable components, manage data through props and state, and efficiently update the DOM when application data changes.

React primarily focuses on the UI layer. It does not prescribe everything an application needs, such as routing, data fetching, or state management; those are commonly handled using additional libraries or framework features such as Next.js.

**Key Points to Remember:**
- Component-based UI library.
- Uses declarative programming.
- Updates the DOM through reconciliation.
- React itself is focused primarily on the UI layer.

---

### Q2. What is a React component?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Understanding of React's fundamental building block.

**Expected Answer:**

A component is a reusable piece of UI that accepts inputs called props and returns React elements describing what should appear on the screen.

Modern React primarily uses function components.

```jsx
function Welcome({ name }) {
  return <h1>Hello, {name}</h1>;
}
```

Here, `Welcome` is a component and `name` is a prop.

**Key Points to Remember:**
- Components promote reuse and separation of concerns.
- Function components are the standard approach today.
- A component can have its own state.
- Components can receive data through props.

---

### Q3. What is JSX?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Whether you understand the syntax used to describe React UI.

**Expected Answer:**

JSX is a syntax extension that allows us to write HTML-like markup inside JavaScript. It is not HTML and browsers do not directly execute JSX. A build tool/compiler transforms JSX into JavaScript calls that create React elements.

```jsx
const element = <h1>Hello</h1>;
```

JSX also allows JavaScript expressions inside `{}`.

```jsx
const name = "Aditya";
const element = <h1>Hello {name}</h1>;
```

**Key Points to Remember:**
- JSX is JavaScript syntax, not HTML.
- It is transformed before execution.
- JavaScript expressions go inside `{}`.
- JSX must generally have one enclosing returned element or fragment.

---

### Q4. What is the difference between props and state?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** One of the most important React fundamentals.

**Expected Answer:**

**Props** are inputs passed from a parent component to a child. They are read-only from the receiving component's perspective.

**State** is data managed by a component that can change over time. Updating state causes React to schedule a re-render.

```jsx
function Counter({ initialCount }) {
  const [count, setCount] = useState(initialCount);
}
```

Here `initialCount` is a prop, while `count` is state.

| Props | State |
|---|---|
| Passed by parent | Managed by component |
| Read-only | Can be updated |
| Used to configure a component | Represents changing data |
| Changes can cause re-render | Updates cause re-render |

**Key Points to Remember:**
- Props flow into a component.
- State belongs to the component/state owner.
- Never directly mutate state.

---

### Q5. What is the Virtual DOM?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you understand React's rendering model rather than repeating "Virtual DOM is faster."

**Expected Answer:**

The Virtual DOM is a lightweight JavaScript representation of the UI that React uses during rendering. When state or props change, React creates a new element tree and compares it with the previous rendered tree.

React then determines the necessary changes and commits the required DOM updates.

The important point is that the Virtual DOM is **not simply a faster DOM**. Its main benefit is that React can reason about UI changes declaratively and efficiently determine what needs to change.

**Key Points to Remember:**
- It is an in-memory representation of UI.
- React compares render results.
- Only necessary DOM changes are committed.
- Do not claim that every Virtual DOM operation is automatically faster than direct DOM manipulation.

---

## 2. Rendering and Internal Working

### Q6. What is reconciliation in React?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of how React determines UI updates.

**Expected Answer:**

Reconciliation is the process React uses to determine how the newly rendered element tree differs from the previous one. Based on those differences, React determines what should be changed in the rendered UI.

For lists, React uses `key` values to help identify which items correspond between renders.

A simplified flow is:

```text
State/Props change
       ↓
Component renders
       ↓
New React element tree
       ↓
Reconciliation
       ↓
Commit required changes
       ↓
Browser displays updated UI
```

**Key Points to Remember:**
- Reconciliation compares render results.
- It is different from the actual DOM commit.
- Keys are important for identifying list elements.

---

### Q7. What causes a React component to re-render?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you understand React's update model.

**Expected Answer:**

A component can re-render when:
- Its state is updated.
- Its parent re-renders and React renders the child again.
- Its context value changes.
- An external store subscribed to by the component changes.

A re-render does **not necessarily mean the DOM is changed**. React can render a component, reconcile the result, and determine that no actual DOM mutation is necessary.

**Key Points to Remember:**
- Re-render ≠ DOM update.
- State updates commonly trigger rendering.
- Parent rendering can cause child rendering.
- `React.memo` can sometimes prevent unnecessary child renders when props are unchanged.

---

### Q8. What is the difference between rendering and committing in React?

**Priority:** 🟠 Important  
**Difficulty:** Tricky

**What the interviewer is testing:** Whether you understand React's rendering pipeline.

**Expected Answer:**

The **render phase** calculates what the UI should look like based on the current props, state, and context.

The **commit phase** applies the necessary changes to the actual host environment, such as the browser DOM.

Conceptually:

```text
Update
  ↓
Render phase → calculate next UI
  ↓
Commit phase → apply necessary DOM changes
```

The render phase should be treated as pure. Side effects should not be performed during rendering.

**Key Points to Remember:**
- Render phase calculates.
- Commit phase applies changes.
- Rendering should remain pure.
- Effects run outside the render calculation.

---

### Q9. Why are keys required when rendering lists?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of list reconciliation and identity.

**Expected Answer:**

Keys give React a stable identity for elements in a list. They help React determine which items were added, removed, or moved between renders.

```jsx
users.map(user => (
  <User key={user.id} user={user} />
))
```

A stable database ID is generally better than using the array index, especially when items can be inserted, deleted, or reordered.

Using indexes as keys can cause incorrect component state to be associated with different list items.

**Key Points to Remember:**
- Keys should be stable and unique among siblings.
- Prefer IDs from the data.
- Avoid random values such as `Math.random()`.
- Index keys are risky for dynamic lists.

---

## 3. State and Hooks

### Q10. What is `useState`, and how does it work?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Core functional-component state management.

**Expected Answer:**

`useState` is a React Hook that lets a function component store state.

```jsx
const [count, setCount] = useState(0);
```

`count` is the current state value and `setCount` requests a state update.

When the update is processed, React can render the component again using the new state.

For updates that depend on the previous value, use the functional form:

```jsx
setCount(prev => prev + 1);
```

**Key Points to Remember:**
- Hooks must be called at the top level of a component or custom Hook.
- Do not directly mutate state.
- Functional updates are useful when the next state depends on previous state.
- State updates are scheduled; don't assume the variable changes immediately within the same execution.

---

### Q11. Why shouldn't you mutate React state directly?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of state updates and immutable data patterns.

**Expected Answer:**

React state should be treated as immutable. Direct mutation can make it harder for React and developers to reason about changes and can lead to stale or inconsistent UI behavior.

Instead of:

```jsx
user.name = "John";
```

create a new object:

```jsx
setUser(prev => ({
  ...prev,
  name: "John"
}));
```

Creating a new reference also makes change detection patterns and memoization more predictable.

**Key Points to Remember:**
- Treat state as immutable.
- Create new objects/arrays when updating.
- Never mutate state and expect React to automatically detect every mutation.

---

### Q12. What is `useEffect` used for?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of effects and dependency behavior.

**Expected Answer:**

`useEffect` is used to synchronize a component with an external system or perform side effects after React has committed an update. Examples include subscriptions, timers, DOM APIs, and fetching data in appropriate cases.

```jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log("tick");
  }, 1000);

  return () => clearInterval(id);
}, []);
```

The returned function is cleanup.

The dependency array controls when the effect is re-run based on the values referenced by the effect.

**Key Points to Remember:**
- Effects are for synchronization/side effects, not general-purpose computation.
- Cleanup prevents leaked subscriptions/timers.
- Dependencies must accurately represent reactive values used by the effect.
- An empty dependency array does not mean "run exactly once in the lifetime of the application"; it is tied to that component instance.

---

### Q13. What is the difference between `useEffect` and `useLayoutEffect`?

**Priority:** 🟠 Important  
**Difficulty:** Tricky

**What the interviewer is testing:** Understanding of browser painting and effect timing.

**Expected Answer:**

`useEffect` generally runs after the browser has had an opportunity to paint, whereas `useLayoutEffect` runs after DOM mutations but before the browser paints.

`useLayoutEffect` is useful when we need to measure or synchronously adjust the DOM before the user sees it, such as measuring an element's dimensions.

Because layout effects can block painting, `useEffect` should be preferred when a layout effect is not necessary.

**Key Points to Remember:**
- `useEffect`: normal side effects.
- `useLayoutEffect`: DOM measurement/layout-sensitive work.
- Don't use layout effects unnecessarily.

---

### Q14. What are the Rules of Hooks?

**Priority:** 🔴 Must Know  
**Difficulty:** Basic

**What the interviewer is testing:** Whether you understand Hook usage constraints.

**Expected Answer:**

The two core rules are:

1. Call Hooks only at the top level, not inside loops, conditions, or nested functions.
2. Call Hooks only from React function components or custom Hooks.

```jsx
// Wrong
if (loggedIn) {
  useEffect(...);
}
```

React relies on Hooks being called in a consistent order across renders.

**Key Points to Remember:**
- Consistent Hook call order is essential.
- Don't conditionally call Hooks.
- Custom Hooks are functions whose names conventionally start with `use`.

---

### Q15. What is `useRef`, and how is it different from `useState`?

**Priority:** 🟠 Important  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of mutable values and DOM references.

**Expected Answer:**

`useRef` returns a mutable object whose `.current` value persists across renders.

```jsx
const inputRef = useRef(null);

<input ref={inputRef} />
```

It is commonly used to access DOM elements or store mutable values that should persist but should not cause a re-render when changed.

| `useState` | `useRef` |
|---|---|
| Updating state schedules a render | Changing `.current` does not trigger a render |
| Represents UI-relevant state | Often stores mutable instance-like data |
| Update through setter | Mutate `.current` |

**Key Points to Remember:**
- Ref values persist between renders.
- Ref changes don't trigger rendering.
- Common uses: DOM access, timer IDs, previous values, mutable instance data.

---

## 4. Data Flow and Component Design

### Q16. What is prop drilling, and how can you avoid it?

**Priority:** 🟠 Important  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of component data flow and state architecture.

**Expected Answer:**

Prop drilling occurs when data has to be passed through several intermediate components that do not actually need the data, only so a deeply nested component can receive it.

Possible solutions include:
- React Context.
- State management libraries such as Redux or Zustand.
- Restructuring components.
- Moving state closer to where it is needed.

However, Context should not automatically replace every use of props. Props are often clearer for direct parent-child communication.

**Key Points to Remember:**
- Props are the normal explicit data-flow mechanism.
- Context is useful for shared values.
- Avoid global state when local state is sufficient.

---

### Q17. What is the Context API?

**Priority:** 🟠 Important  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of React's built-in shared-data mechanism.

**Expected Answer:**

Context allows a value to be made available to components deeper in the component tree without manually passing it through every intermediate component.

Typical examples include:
- Theme.
- Locale.
- Authentication information.
- Application-wide configuration.

```jsx
const ThemeContext = createContext();

<ThemeContext.Provider value={theme}>
  <App />
</ThemeContext.Provider>
```

Consumers can read the context using `useContext`.

**Key Points to Remember:**
- Avoids certain forms of prop drilling.
- Context is not automatically a complete state-management solution.
- Consumers can re-render when the context value they consume changes.

---

### Q18. What is a controlled component?

**Priority:** 🟠 Important  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of form state management.

**Expected Answer:**

A controlled form element gets its current value from React state, and changes are handled through React event handlers.

```jsx
const [email, setEmail] = useState("");

<input
  value={email}
  onChange={e => setEmail(e.target.value)}
/>
```

React becomes the source of truth for the input value.

An uncontrolled input instead keeps its value primarily in the DOM and can commonly be accessed through a ref.

**Key Points to Remember:**
- Controlled = React state is the source of truth.
- Useful for validation and dynamic form behavior.
- Uncontrolled inputs can be simpler for some forms and integrations.

---

## 5. Performance and Optimization

### Q19. What is `React.memo`?

**Priority:** 🟠 Important  
**Difficulty:** Tricky

**What the interviewer is testing:** Whether you understand memoization rather than blindly applying optimization.

**Expected Answer:**

`React.memo` memoizes a function component so React can skip re-rendering it when its props are considered unchanged.

```jsx
const UserCard = React.memo(function UserCard({ user }) {
  return <div>{user.name}</div>;
});
```

By default, React compares props using shallow comparison semantics.

It is useful when a component renders frequently and receives stable props, but it is not automatically beneficial everywhere. Memoization has its own comparison cost and can add complexity.

**Key Points to Remember:**
- It optimizes component rendering, not application logic in general.
- Stable prop references matter.
- Don't use it everywhere without measuring.

---

### Q20. What are `useMemo` and `useCallback`?

**Priority:** 🟠 Important  
**Difficulty:** Intermediate

**What the interviewer is testing:** Understanding of memoization and function identity.

**Expected Answer:**

`useMemo` caches the result of a calculation between renders when its dependencies have not changed.

```jsx
const filtered = useMemo(
  () => expensiveFilter(items, query),
  [items, query]
);
```

`useCallback` caches a function reference.

```jsx
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);
```

They are useful in specific performance-sensitive situations, especially when expensive calculations or stable references matter for memoized children.

**Key Points to Remember:**
- `useMemo` → memoizes a value/result.
- `useCallback` → memoizes a function reference.
- They are optimizations, not defaults.
- Incorrect dependencies can create stale values/functions.

---

### Q21. What causes unnecessary re-renders in React, and how can you optimize them?

**Priority:** 🔴 Must Know  
**Difficulty:** Tricky

**What the interviewer is testing:** Practical React performance understanding.

**Expected Answer:**

Common causes include:
- State being stored too high in the component tree.
- Parent components rendering frequently.
- Passing newly created objects/functions to memoized children.
- Large lists being rendered unnecessarily.
- Unnecessary context updates.
- Expensive calculations during render.

Possible optimizations:
- Keep state close to where it is used.
- Split components.
- Use `React.memo` when appropriate.
- Use `useMemo`/`useCallback` selectively.
- Virtualize very large lists.
- Avoid unnecessary context updates.
- Measure performance before optimizing.

**Key Points to Remember:**
- First identify the bottleneck.
- Re-rendering is not automatically a performance bug.
- Optimization should be evidence-driven.

---

## 6. Events, Effects, and Data Fetching

### Q22. How does event handling work in React?

**Priority:** 🟠 Important  
**Difficulty:** Basic

**What the interviewer is testing:** Practical understanding of React event handling.

**Expected Answer:**

React uses event handlers written as props such as `onClick`, `onChange`, and `onSubmit`.

```jsx
<button onClick={handleClick}>
  Save
</button>
```

We pass a function rather than calling it immediately.

```jsx
// Correct
<button onClick={handleClick} />

// Usually wrong
<button onClick={handleClick()} />
```

React's event system provides a consistent programming model across browsers.

**Key Points to Remember:**
- Event names use camelCase.
- Pass a function.
- For arguments, use a function wrapper: `onClick={() => deleteUser(id)}`.
- `preventDefault()` can prevent browser default behavior.

---

### Q23. How would you fetch API data in a React application?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you can implement normal React application behavior.

**Expected Answer:**

A simple client-side implementation can use `useEffect` with an asynchronous function:

```jsx
useEffect(() => {
  let ignore = false;

  async function loadUsers() {
    const response = await fetch("/api/users");
    const data = await response.json();

    if (!ignore) {
      setUsers(data);
    }
  }

  loadUsers();

  return () => {
    ignore = true;
  };
}, []);
```

In a production application, the exact approach depends on the architecture. Libraries such as TanStack Query can manage caching, retries, loading states, synchronization, and request lifecycles more effectively than hand-written effects.

**Key Points to Remember:**
- Handle loading, success, and error states.
- Consider cancellation/race conditions.
- Avoid unnecessary duplicate requests.
- Client-side fetching is not the only approach; frameworks such as Next.js provide server-side/data-fetching patterns.

---

## 7. Architecture and Practical Decisions

### Q24. What is the difference between React and a framework like Next.js?

**Priority:** 🔴 Must Know  
**Difficulty:** Intermediate

**What the interviewer is testing:** Whether you understand where React ends and the framework begins.

**Expected Answer:**

React is primarily a library for building UI. Next.js is a React framework that provides additional application-level capabilities and conventions.

Depending on the version and architecture, Next.js can provide features such as:
- Routing.
- Server-side rendering.
- Static generation.
- Server Components.
- Server-side data access.
- API/route handlers.
- Build and deployment conventions.

So React can be used as the UI foundation, while Next.js provides a broader application framework around React.

**Key Points to Remember:**
- React ≠ Next.js.
- Next.js uses React.
- Framework features can reduce the need to assemble many separate libraries.

---

### Q25. What are common React mistakes or anti-patterns you would avoid?

**Priority:** 🔴 Must Know  
**Difficulty:** Tricky

**What the interviewer is testing:** Whether you have practical experience rather than only theoretical knowledge.

**Expected Answer:**

Some common mistakes are:

1. Mutating state directly.
2. Using unstable/random keys for lists.
3. Calling Hooks conditionally.
4. Putting every piece of state into global state.
5. Using `useEffect` for calculations that could happen during rendering.
6. Ignoring effect dependencies.
7. Overusing `useMemo`, `useCallback`, or `React.memo`.
8. Creating unnecessarily large components.
9. Not handling loading and error states.
10. Performing expensive work on every render without considering its cost.

A good React design keeps state close to where it is needed, uses components with clear responsibilities, and optimizes based on measured performance problems.

**Key Points to Remember:**
- Correctness first, optimization second.
- Keep components and state responsibilities clear.
- Treat effects as synchronization tools, not a replacement for normal JavaScript logic.
- Stable identity is important for list rendering and memoization.

---

# Final Revision Section

## 1. 🔴 Must-Know Questions

Before a React interview, make sure you can answer these without hesitation:

1. What is React?
2. What are components?
3. What is JSX?
4. Props vs state?
5. What is the Virtual DOM?
6. What is reconciliation?
7. What causes a component to re-render?
8. What are keys and why are they important?
9. How does `useState` work?
10. Why shouldn't state be mutated directly?
11. How does `useEffect` work?
12. What are the Rules of Hooks?
13. How does Context work?
14. How would you fetch API data?
15. React vs Next.js?
16. What common React mistakes should be avoided?

---

## 2. ⚠️ Tricky Concepts

These are particularly useful for detecting whether a candidate genuinely understands React:

### Re-render ≠ DOM update

A component can render again while React determines that no actual DOM mutation is necessary.

### Virtual DOM ≠ automatically faster DOM

The important benefit is React's declarative rendering and reconciliation model, not simply that a JavaScript object is faster than the DOM.

### `useEffect` ≠ "code that runs after every render"

Its purpose is synchronization with external systems, and its dependencies determine when React re-runs it.

### `useMemo` ≠ always a performance improvement

Memoization has costs and should be used when it addresses a real performance problem.

### `useRef` ≠ another form of state

Changing `ref.current` does not trigger a render.

### Index as key ≠ always wrong

It can be acceptable for a static list that never changes order, but it becomes risky when items can be inserted, removed, or reordered.

### Props are not "immutable data everywhere"

A component should not mutate its received props. The parent owns the source of that data.

### Context ≠ automatically global state management

Context provides a mechanism for passing values through a tree. It does not by itself provide all the features of a dedicated state-management solution.

---

# 3. 🧠 Quick Revision Sheet

## React Mental Model

```text
Props / State / Context
          ↓
       Render
          ↓
New React element tree
          ↓
   Reconciliation
          ↓
      Commit
          ↓
     DOM update
```

## Core Concepts

| Concept | Remember |
|---|---|
| Component | Reusable UI unit |
| JSX | JavaScript syntax for describing UI |
| Props | Parent → child inputs |
| State | Component-managed changing data |
| Virtual DOM | In-memory UI representation |
| Reconciliation | Determines differences between render results |
| Key | Stable identity for list elements |
| Hook | Function providing React features to function components |
| Context | Pass shared values through component tree |
| Effect | Synchronize with external systems |
| Ref | Persistent mutable value/DOM reference |
| Memoization | Avoid repeated calculations/renders when appropriate |

## Important Hooks

```text
useState     → component state
useEffect    → external synchronization / side effects
useRef       → persistent mutable value / DOM reference
useContext   → consume Context
useMemo      → memoize calculated value
useCallback  → memoize function reference
```

## Performance

Think in this order:

```text
Measure
  ↓
Find bottleneck
  ↓
Reduce unnecessary work
  ↓
Memoize / virtualize / restructure where appropriate
  ↓
Measure again
```

---

# 4. 🔑 Important Terms

You should immediately recognize and be able to explain:

- Component
- Function component
- JSX
- Props
- State
- Hook
- `useState`
- `useEffect`
- `useRef`
- `useContext`
- `useMemo`
- `useCallback`
- Virtual DOM
- Reconciliation
- Render phase
- Commit phase
- Re-render
- DOM update
- Key
- Controlled component
- Uncontrolled component
- Prop drilling
- Context
- `React.memo`
- Memoization
- Dependency array
- Effect cleanup
- Immutable state
- Lifting state up
- Component composition
- Client-side rendering
- Server-side rendering
- React framework
- Next.js

---

# 5. 🎯 Interview Tips

### 1. Don't stop at definitions

If asked:

> "What is `useEffect`?"

Don't just say:

> "It is used for side effects."

Add a concrete example such as subscriptions, timers, or synchronizing with an external system.

### 2. Be careful with "Virtual DOM is faster"

This is a common beginner answer. A better explanation focuses on React's rendering and reconciliation model.

### 3. Know what happens after `setState`

A strong fresher should be able to explain:

```text
setState
  ↓
Update is scheduled
  ↓
Component may render again
  ↓
React reconciles the result
  ↓
Necessary changes are committed
```

### 4. Expect follow-up questions

If you say:

> "I used `useEffect` to fetch API data."

Expect:

- Why `useEffect`?
- What happens if the dependency array is omitted?
- What does `[]` mean?
- How do you handle errors?
- What about race conditions?
- How would you cache the response?

Only claim technologies you can explain at this level.

### 5. Connect answers to your actual project

If React is on your resume, be prepared to explain exactly:

- How your components were structured.
- Where state lived.
- How data flowed between components.
- How API calls were handled.
- How forms were implemented.
- How errors/loading states were handled.
- Why you chose a particular state-management approach.
- What performance issues you encountered, if any.
- How the frontend communicated with your backend.

### 6. Don't over-engineer

For a fresher interview, saying:

> "I used Redux because every application needs Redux"

is weaker than:

> "I kept local state for component-specific data and introduced shared state only where multiple unrelated components needed the same data."

That demonstrates actual engineering judgment.

---

# 6. ❓ Self-Test

Try answering these aloud without looking at the guide:

1. Explain React in 30 seconds.
2. Explain the difference between props and state.
3. Explain what happens after a state update.
4. Explain Virtual DOM and reconciliation.
5. Why are keys necessary?
6. What causes a component to re-render?
7. Why should state not be mutated directly?
8. Explain `useState`.
9. Explain `useEffect` and its dependency array.
10. What is effect cleanup?
11. Why can't Hooks be called conditionally?
12. `useRef` vs `useState`?
13. `useMemo` vs `useCallback`?
14. What is prop drilling?
15. Context vs props?
16. Controlled vs uncontrolled components?
17. What is `React.memo`?
18. How would you optimize a slow React component?
19. How would you fetch data from an API in React?
20. React vs Next.js?
21. Why is using array index as a key sometimes problematic?
22. Why can unnecessary `useEffect` usage be a problem?
23. What is the difference between render and commit?
24. What React performance optimizations have you actually used?
25. If React is on your resume, explain exactly how you used it in your project.

---

# Final Interview Target

For a **fresher Software Engineering interview**, you should aim to reach this level:

> **Definition → How it works → Small example → Trade-off/pitfall**

For example, when asked about `useMemo`, don't stop at:

> "It memoizes a value."

A stronger answer is:

> "`useMemo` caches the result of a calculation between renders until its dependencies change. It can reduce repeated expensive calculations, but I wouldn't use it everywhere because memoization itself has overhead and can make code more complex. I'd use it when the calculation is expensive or when stable derived values are important for an optimization such as a memoized child."

That style demonstrates **understanding rather than memorization**, which is exactly what an interviewer is trying to assess when they see React on a fresher's resume.
