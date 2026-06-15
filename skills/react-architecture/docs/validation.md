# Validation

Validation lives in root `validation/`.

```txt
validation/zod-schemas/  Form or payload schemas.
validation/rules/        Validation or business rules for valid/allowed behavior.
```

- Naming: files under `zod-schemas/` must start with a verb when tied to an action and end with `schema`, such as `store-pantry-item-schema.ts` options `store`, `update`, `destroy`, or feature specific `upload` or something else.
- Zod schema files must export only one schema and nothing else.
- Zod schema files must not contain functions; put custom checks in `validation/rules/`.
- Naming: files under `rules/` should be named after the rule, such as `non-restrictive-name.ts`.
- Rule files must export only one function named `validate`.
- `validate` must return `true` or `false`.
- All helper functions inside rule files must stay local.
- Run Zod schemas and validation rules at form submission.
