---
name: react-architecture
description:
    This describes the pattern used in this project for adding new features and refactoring old ones.
    Use this skill when we are discussing or modifying code for a new frontend feature or editing old ones.
---

**Use this skill when**:

- Adding a new frontend feature or changing existing code.
- Deciding where frontend code should live.
- Reviewing frontend code

**Do not use this skill when**:

- The task is only a shell/git/devops action and does not involve backend code structure.
- The task is only formatting, typo fixing, or tiny text edits with no architecture or layer decision.
- Working on backend code

# React Architecture

Use this file only to decide where logic belongs. After choosing the path, read the linked doc for detailed conventions.

- `components/{feature}/` - UI that renders views, receives props, and calls callbacks - check `docs/components.md`.
- `hooks/{feature}/` - React behavior using state/effects, app hooks, queries, navigation, animation, or local UI coordination - check `docs/hooks.md`.
- `model/{feature}/` - pure TypeScript feature logic for valid data - check `docs/model.md`.
- `api/` - HTTP clients, provider clients, endpoint files, or HTTP transport - check `docs/api.md`.
- `queries/` - TanStack Query keys, query options, mutation options, invalidation, or prefetching - check `docs/queries.md`.
- `validation/zod-schemas/` and `validation/rules/` - form/payload schemas or validation/business rules - check `docs/validation.md`.
- `lib/` - facades around app-wide libraries - check `docs/lib-and-utils.md`.
- `utils/` - generic reusable helpers that are not facades and not feature-specific - check `docs/lib-and-utils.md`.
- `stores/` - global client state, normally Zustand.

- If a folder has `{feature}`, group by feature there. If it does not have `{feature}`, keep files directly under that root boundary.