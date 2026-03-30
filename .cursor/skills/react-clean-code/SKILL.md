---
name: react-clean-code
description: React and TypeScript clean-code standards for component architecture, state, effects, and performance. Use when writing, refactoring, or reviewing React/TS code, components, hooks, and frontend logic.
---

# React & TypeScript Development Standards

## 1. Clean Code

- **Early Returns:** Avoid nested `if` statements. Use guard clauses (e.g., `if (!condition) return;`) so the happy path stays at the lowest indentation level.
- **Simplicity:** Favor simplicity over cleverness. Avoid recursion, deep callbacks, and complex abstractions unless strictly necessary.
- **Method Limits:** Methods/functions should have a maximum of 4 parameters.
- **State Validation:** Validate state before executing business actions. Example: `cancel()` must verify cancellation is allowed, otherwise throw.
- **Self-Documenting Code:** Prefer clear naming that explains intent.
- **Naming:** Use full descriptive words; avoid abbreviations (e.g., `subscription`, not `sub`).

## 2. Comments

- Never add comments to user code.

## 3. Component Architecture

- **Single Responsibility:** Each component or hook owns only its scope. Do not leak parent logic into children.
- **JSX Purity:** Keep heavy logic out of JSX; prepare values before `return`.
- **Import Patterns:** Use named imports (e.g., `import { useState } from "react"`), and prefer path aliases (e.g., `@/components/...`) when available.

## 4. `useEffect` Rules

Avoid `useEffect` for:

1. Derived state (compute in render or `useMemo`).
2. Filtering/transforming data (compute directly).
3. Resetting state (use `key` remount strategy).
4. User events (handle in event handlers).
5. Local storage sync (read during initialization, write in the updating handler).
6. Prop syncing (prefer `key` strategy).

Use `useEffect` only for:

- External system synchronization (browser APIs, third-party widgets).
- Data fetching when not using React Query/SWR.
- Subscriptions/observers/sockets with cleanup.

## 5. State Management & Refs

- **Locality:** Keep state as close as possible to where it is used.
- **Grouping:** Group related state into one object or `useReducer` when updates are coupled.
- **Global State:** Use Zustand for shared/global state.
- **Refs vs State:** Use `useRef` for values that should not trigger re-renders.
- **Imperative Handles:** Keep child internals in the child; expose only a narrow imperative API when needed.

## 6. Performance

- **Structural Optimization:** Prevent re-renders by moving state-heavy logic into smaller components.
- **Variable Placement:** Move scope-independent values/functions outside components.
- **React Compiler:** Avoid manual `memo`/`useCallback`/`useMemo` unless there is a concrete need.
- **Inline Functions:** Avoid passing inline functions in JSX.

## 7. Styling & TypeScript

- **Styling:** Never use inline styles.
- **Type Reuse:** Move reusable types/interfaces into dedicated `.types.ts` files.
- **Function Style:** Prefer `function` declarations unless the project convention differs.

## 8. Error Handling

- **Boundaries:** Use React Error Boundaries for resilient UI fallbacks.
- **Async Errors:** Handle async failures with `try/catch` and user-friendly error states.
