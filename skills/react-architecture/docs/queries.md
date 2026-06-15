# Queries

Query files contain TanStack Query keys, reusable query options, and reusable mutation options. They should be grouped like API files.

## File organization

- Keep normal CRUD query options and mutation options for one resource in one file.
- If an endpoint is special, such as exporting data, create a separate queries file for that endpoint only.
- Naming: files must end with `queries`, such as `pantry-queries.ts` or `export-pantry-queries.ts`.
- Keep queries centered around a resource. Do not create global catch-all query files.
- Export query key constants, query option functions, and mutation option functions.

## Query keys

- Put shared query keys in resource-level exported constants so options, invalidation, cache updates, and prefetching reuse the same key source.
- Query keys should be factories, not scattered inline arrays. Include every variable that changes the fetched data in the key.

## Query options

- Extract reusable query option functions for each read operation. The function should return `queryKey`, `queryFn`, and default cache behavior such as `staleTime` when needed.
- Use TanStack Query's `queryOptions` helper for reusable query option functions so TypeScript keeps the key, query function, and returned data linked.
- Query option functions should be plain functions, even when they currently take no arguments. This keeps the shape consistent when the query later needs an ID, filter, or parameter.
- Reusable query option functions must be usable by `useQuery`, `useSuspenseQuery`, and non-component prefetching code.
- Use the same query option function for route loaders, `queryClient.ensureQueryData`, `queryClient.prefetchQuery`, `useQuery`, and `useSuspenseQuery`.
- Query functions must return a promise that resolves to data or rejects with an error. Return `null` for an intentional empty result, not `undefined`.

## Component usage

- Prefer consuming query option functions directly with `useQuery` or `useSuspenseQuery` at the call site.
- Only create a query hook when it adds meaningful behavior beyond calling `useQuery` with options.
- Prefer fetching route-level data in the navigation layer when the router supports loaders or server components. Components should receive already-loaded data through suspense/query cache instead of owning every pending, error, and redirect branch.
- Prefer `useSuspenseQuery` when a route loader, server component, or suspense boundary is responsible for the loading and error states.
- Keep component-level `useQuery` for genuinely client-local or interaction-driven reads where the component owns the loading state.
- It is fine for multiple components to consume the same query independently. Let TanStack Query share cached data instead of pushing query results through unrelated props.
- Keep screen components focused on rendering. Move data loading coordination to query hooks or native navigation/suspense boundaries, not ad hoc `isPending` and `isError` branches spread through UI components.
- Do not copy query option objects between components. Reuse the option function; the repeated `useQuery` or `useSuspenseQuery` call is fine.

## Mutations

- Extract reusable mutation option functions for each write operation. The function should return `mutationFn` and shared mutation behavior when needed.
- Use TanStack Query's `mutationOptions` helper for reusable mutation option functions when available.
- Give related mutations a `mutationKey` when the app needs mutation filtering, shared pending indicators, or `useMutationState`.
- Prefer consuming mutation option functions directly with `useMutation` at the call site.
- Only create a mutation hook when it adds meaningful app behavior beyond calling `useMutation` with options, such as optimistic updates, toasts, navigation side effects, or repeated cache coordination.
- Mutation options or mutation hooks should invalidate or update cache entries through the shared resource query keys.
- Await invalidations in mutation callbacks when the UI should stay pending until fresh data has been requested.
- Prefer invalidating or updating the narrowest stable key that represents the changed data. Use broader resource keys only when multiple cached views are truly affected.

## Boundary

- Do not use TanStack Query for client-only state. Use it for server state: fetching, caching, synchronizing, and invalidating data owned outside the UI.

Example:

```txt
queries/pantry-queries.ts
  pantryQueryKeys
  pantryIndexQueryOptions()       Query options for listing pantry items.
  pantryShowQueryOptions(id)      Query options for one pantry item.
  storePantryItemMutationOptions() Mutation options for creating a pantry item.
```
