# API

API files implement HTTP calls and should be grouped by provider, then like controllers.

- Put the generic shared HTTP transport at `api/http-client.ts`.
- Put each configured provider client inside its provider folder, such as `api/food-cal/food-cal-client.ts` or `api/open-food-facts/open-food-facts-client.ts`.
- Put resource endpoint files inside the provider folder that owns those endpoints.
- Keep normal CRUD methods for one provider resource in one file.
- If an endpoint is special, such as exporting data, create a separate API file for that endpoint only.
- Naming: endpoint files must end with `api`, such as `api/food-cal/pantry-api.ts` or `api/food-cal/export-pantry-api.ts`.
- Naming: provider clients should end with `client`, such as `api/food-cal/food-cal-client.ts`.
- Naming: resource API files should export controller-like functions: `index`, `store`, `show`, `update`, and `destroy` when those endpoints exist.
- Naming: special API files should export one function named `invoke`.
- Import API files as namespaces so controller-like function names stay clear at call sites.

Examples:

```txt
api/http-client.ts
api/food-cal/food-cal-client.ts
api/food-cal/recipe-api.ts
api/food-cal/user-api.ts
api/food-cal/food-log-api.ts
api/open-food-facts/open-food-facts-client.ts
api/open-food-facts/product-api.ts
```
