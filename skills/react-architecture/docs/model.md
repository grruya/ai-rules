# Model

Use `model/{feature}/` for pure TypeScript feature logic.

- No React imports.
- No React Native imports.
- No API calls.
- No TanStack Query.
- No Zod.
- Model files expect valid data. Run Zod schemas and validation rules before calling model functions or mutations.
- Model files are capability modules, not entity buckets. Name each file after a specific feature mechanic or capability.
- Better split into more files than use a broad name to fit more functions.
- Model files may export multiple functions when they belong to the same capability.
- Function names must be understandable by reading what they do.
- All functions in model files must be exported; do not use local helper functions.

Examples:

```txt
model/autocomplete/autocomplete-suggestions.ts
model/autocomplete/autocomplete-keyboard.ts
model/whiteboard/whiteboard-history.ts
model/whiteboard/whiteboard-selection.ts
model/food-log/food-log-row-editing.ts
```
