---
name: react-patterns
description: Builds clean, maintainable React and TypeScript code by enforcing strong component boundaries, disciplined state/effect usage, and practical performance patterns. Use when writing, refactoring, or reviewing React components, hooks, and frontend logic for clarity, scalability, and fewer re-render bugs.
---

# React & TypeScript Development Standards

## 1. Component Architecture

- **Single Responsibility:** Each component or hook owns only its scope. Do not leak parent logic into children.
- **JSX Purity:** Keep heavy logic out of JSX; prepare values before `return`.
- **Import Patterns:** Use named imports (e.g., `import { useState } from "react"`), and prefer path aliases (e.g., `@/components/...`) when available.

## 2. `useEffect` Rules

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

## 3. State Management & Refs

- **Locality:** Keep state as close as possible to where it is used.
- **Grouping:** Group related state into one object or `useReducer` when updates are coupled.
- **Global State:** Use Zustand for shared/global state.
- **Refs vs State:** Use `useRef` for values that should not trigger re-renders.
- **Imperative Handles:** Keep child internals in the child; expose only a narrow imperative API when needed.

## 4. Performance

- **Structural Optimization:** Prevent re-renders by moving state-heavy logic into smaller components.
- **Variable Placement:** Move scope-independent values/functions outside components.
- **React Compiler:** Avoid manual `memo`/`useCallback`/`useMemo` unless there is a concrete need.
- **Inline Functions:** Avoid passing inline functions in JSX.

## 5. Styling & TypeScript

- **Styling:** Never use inline styles.
- **Type Reuse:** Move reusable types/interfaces into dedicated `.types.ts` files.
- **Function Style:** Prefer `function` declarations unless the project convention differs.

## 6. Error Handling

- **Boundaries:** Use React Error Boundaries for resilient UI fallbacks.
- **Async Errors:** Handle async failures with `try/catch` and user-friendly error states.
