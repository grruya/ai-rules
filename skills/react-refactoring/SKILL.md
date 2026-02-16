---
name: react-refactoring
description: Acts as a senior React refactoring expert, enforce clean code (early returns, no dead code), eliminates unnecessary useEffect in favor of derived state/key-props, optimizes performance via component structure over memoization, and ensures strict TypeScript type-hinting and memory leak cleanup.
---

## Clean Code

1. Avoid Nested Ifs – Enforce Early Returns

- Avoid nesting multiple if statements.
- Use early returns or negated conditions (if (!condition) return;) to reduce complexity and improve readability.

2. Smart Comments

- Don't comment on what the code does - make the code self-documenting
- Use comments to explain why something is done a certain way
- Document APIs, complex algorithms, and non-obvious side effects
- Don't comment what could be implemented in future or the steps u took or any od your toughts

3. Meaningful Names

- Avoid abbreviations unless they're universally understood
- The naming of function and variables should be good so the code could be understood by just reading it

4. Code Quality Maintenance

- Write code that someone else (or future you) can easily read, understand, and modify without guessing.
- Make the code easier to understand and read
- If possible make the code shorter then u found it
- Remove and dead code or redundant checks or fallbacks

5. Simplicity over cleverness – Avoid recursion, callbacks, deep nesting, try-catch pyramids, and overly clever abstractions.

6. Abstraction rule – Only abstract logic into separate functions when you need to reuse that logic in two or more places.

7. Prefer longer functions – Prioritize readability and linear flow over fragmented, tiny functions.

8. Defensive programming – Every function should include defensive checks if type-hinting properties and return values cannot fully validate the data.

9. Detailed type-hinting – Always type-hint collections, arrays, and other lists in detail, specifying exactly what they should contain

10. Narrow scope – If a variable can be moved locally into a function, loop, or specific block, do it; keep logic as close as possible to where it is used to narrow the scope.

---

## When to use `useEffect`

### DON'T use `useEffect` for:

#### 1. Calculating derived state

If something can be calculated from existing state or props, compute it directly during render. Do not store it in state and do not update it inside an effect.

#### 2. Filtering or transforming data

If the work is cheap, compute it directly during render.  
If the work is expensive, wrap it in a memoization hook to avoid recalculation — not in an effect.

#### 3. Resetting state when props change

Do not use effects to manually reset state when a prop changes.  
Instead, allow React to re-mount the component by providing a unique key, ensuring a clean internal state.

#### 4. Handling user events

Actions triggered by the user (clicks, submits, typing) should run directly in the event handler function, not inside an effect that reacts afterward.

#### 5. Reading from Local Storage

LocalStorage is synchronous. You can read it before initializing state or directly inside the render logic without an effect.

#### 6. Computing initial state

If you can compute something immediately and synchronously before calling `useState`, do it there instead of using an effect.

#### 7. Updating Local Storage when state changes

If you already know exactly when an item is added or removed, update LocalStorage in the event handler itself, not in an effect.

#### 8. Listening for prop changes passed from parent

- Scenario: `<App />` has a state `chosenCount` (initially 0) and renders `<Counter />` with `initialCount={chosenCount}`.
- Inside `<Counter />`, you have a `count` state initialized from `initialCount` and a `useEffect` listening for changes to `initialCount` to update `count`.
- You can **avoid this `useEffect` entirely** by using:

```jsx
<Counter key={chosenCount} initialCount={chosenCount} />
```

- React will throw away the old `<Counter />` instance and mount a new one whenever `chosenCount` changes.
- This ensures the new `Counter` renders with the updated value, eliminating the need for a `useEffect`.

### DO use `useEffect` for:

- Synchronizing with external systems, such as browser APIs, third-party scripts, widgets, and imperative DOM interactions
- Data fetching (unless using framework tools like React Query, SWR, or built-in data loaders)
- Subscribing to external stores, sockets, observers, or events (prefer stable APIs like `useSyncExternalStore` when possible)
- Running logic that must occur _after_ the component has rendered and cannot safely run during the render phase

**Always include cleanup** when subscribing, fetching, or performing async work to avoid race conditions and memory leaks.

---

## State Management

- If state is related, group it in one state
- Use Zustand for shared/global state
- Keep state as close to where it's used as possible
- Avoid prop drilling through proper state management

---

### When to use a reducer

- Many state updates affecting the same piece of state.
- State logic is spread across multiple event handlers.
- You want clearer separation between what happened (action) and how state changes (reducer).
- You want pure functions for easier testing/debugging.

---

## useRef

### When to use it

- Try using ref whenever you can instead of useState. For example, if you just need a value from an input, use ref and get its value via ref instead of state.
- If you have a ref playerName tied to an input and you manage its value with the ref, that's fine. But if you want to display that value in JSX with playerName?.current?.value, changing it via the ref won't trigger a re-render, so the displayed value will not update. That's why you need to use some state for enteredPlayerName.

### Reusable code

- Make sure to utilize useImperativeHandle. This is a hook for useRef that lets you keep component logic where it belongs.
- For example, if you have a parent that triggers a modal to open, but the modal is a separate component, the parent can pass a ref. You don't want the open/close logic in the parent—this is when useImperativeHandle is useful. You can define an open function in the child, and just call that function from the parent. The logic of the open function stays in the child, and the parent simply invokes it.

---

## React Performance Optimizations

### General performance tips

- If somthing can be stored with a normal variable or outside of the component function to achive the desired behavior do it that way instead of useState or useRef
- Dont pass inline functions bcs they trigger a rerender
- If possible move functions outside of the component function
- U dont need to use this: memo, useCallback, useMemo bcs why use react compiler now that does that for us

### Clever structuring

- You can prevent rerender by putting code into separate components.
- Example: You have a parent `<App />` and inside it a `<Counter />` (with some buttons). Also in `<App />`, you have a `useState` with an input and a button to set a name. When you set the name, `<App />` rerenders and `<Counter />` also rerenders because it's under `<App />`.
- Instead of compiler putting `memo` on `<Counter />`, you could move the input and button into a separate component like `<SetName />`. Then when the state in `<SetName />` changes, only `<SetName />` rerenders.

---

## Error Handling

- Implement Error Boundaries
- Handle async errors properly
- Show user-friendly error messages
- Implement proper fallback UI
- Log errors appropriately
- Handle edge cases gracefully

---

## Preferences

- Never use `React.hookName` directly; import hooks like `import { hookName } from "react"`
- When importing components, use `@/path-name`
- Follow the current conventions of code base i prefer using normal functions over arrow but if the code base uses arrow use those then
- Never use inline styles, use tailwind in className or make a css class and put it in className
- Make sure if possible to make components know only their concernes like if u have a Wizard.jsx and a child Form.jsx the form compoent should not
  have any logic in it that will manage the parent wizard it should only handle his form logic no other concernes, functions or logic
- Never do extensive logic in jsx

---

## TypeScript

- If types or interfaces declared in components could be reused in future, move them to a separate `.types.ts` file

---

## Eliminating Memory Leaks

- Event Listener Management: Ensure all global event listeners are detached when a component unmounts to prevent background processing of dead components.
- Interval & Timer Cleanup: Always clear `setInterval` or `setTimeout` calls to avoid logic running indefinitely in the background.
- Async Operation Cancellation: Cancel pending network requests or promises if the user navigates away before the operation completes.
- `useEffect` Cleanup Patterns: Strictly utilize the return function in `useEffect` hooks to reset state, unsubscribe from streams, and clear memory allocations.
