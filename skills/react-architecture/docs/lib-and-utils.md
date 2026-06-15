# Lib And Utils

Use `lib/` for facades around libraries.

Examples:

```txt
lib/query-client.ts
lib/secure-storage.ts
```

Use `utils/` for generic reusable functions that are not facades and not feature-specific.
All functions in that files need to be exported no local ones.

Examples:

```txt
utils/colors.ts
utils/weight-unit.ts
utils/date.ts
```
