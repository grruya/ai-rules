# Hooks

Use `hooks/{feature}/` for React behavior.

- Hooks can use React state/effects, app hooks, queries, mutations, navigation, animation, toasts, and local UI behavior.
- Do not create a custom hook just to wrap one query hook unless it adds meaningful UI behavior.
- Hook files export exactly one runtime function whose name starts with `use`.
