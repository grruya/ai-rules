---
name: zustand
description: Structures Zustand stores for performance and maintainability by promoting atomic selectors, stable action APIs, and small focused stores that minimize unnecessary re-renders. Use when creating or refactoring Zustand state management, store APIs, selectors, or persistence patterns in React apps.
---

These best practices aim to improve the performance, maintainability, and clarity of your Zustand code.

- **Export Custom Hooks Only:**
    - Instead of exporting the main `useTodoStore` hook, create separate custom hooks that encapsulate selector logic.
    - Example: `useTodos`, `useIsSubscribed`.
    - Avoids unnecessary re-renders by preventing components from subscribing to the entire store.
    - Centralizes selector logic for easier maintenance.

- **Use Atomic Stable Selectors for State:**
    - Avoid creating object selectors (e.g., `{ todos, isSubscribed }`) because Zustand uses strict equality checks.
    - Use separate, atomic hooks for individual state values (e.g., `useTodos`, `useIsSubscribed`).
    - If object selectors are necessary, use `useShallow` for shallow comparison.

- **Separate Actions from State:**
    - Create a single, stable `actions` object to hold all functions that modify the state.
    - Example: `{ addTodo, removeTodo, toggleIsSubscribed }`.
    - Actions should reflect business logic/events, not generic setters.
    - This pattern is an exception to the "atomic selector" rule, but acceptable because the `actions` object itself is stable.

- **Model Actions as Events:**
    - Name actions to reflect what happens in the application, not just how the state changes.
    - Example: `addTodo` instead of `setTodo`.
    - Keeps business logic within the store, not scattered across components.

- **Keep Stores Small and Focused:**
    - Create multiple small, focused stores instead of one large store.
    - Example: Separate stores for todos and subscription status if they are unrelated.
    - Reduces the risk of unnecessary re-renders.
    - Improves code organization and testability.
    - Consider "slices" within a single bound store as an alternative.

- **Utilize Middleware:**
    - Enhance Zustand with middleware for common tasks.
    - **`immer`**: Simplifies immutable updates.
    - **`persist`**: Automatically persists state to local storage.

    - **`devtools`**: Integrates with Redux DevTools for debugging.
