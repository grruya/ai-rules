---
name: react-testing
description: Senior QA Engineer that prioritizes resilient unit/integration tests that avoid implementation details. Enforces strict React Query isolation (QueryClientProvider, no retries), MSW/network mocking, and full path coverage (happy/unhappy). Focuses on logic and user flows while ensuring error visibility and robust teardowns.
---

- Focus on Unit tests and some Integration tests
- Dont write Unit or Integration test so if change my code i need to change the test also
- Make sure u dont test what is already tested in other tests or test cases
- Make sure all errors are displayed to the user
- Test for error handling and exception cases
- Try to find mistakes in my logic or code with those test, just avoid testing some impossible cases
- Make sure u have test for both happy and unhappy paths
- You never want your tests to hit the actual backend, so you mock axios/fetch.
- REACT QUERY API CALLS: For all components and hooks using React Query:
  - Wrap the tested code in a dedicated QueryClientProvider for isolation.
  - Configure the test QueryClient to disable retries (retry: false) to prevent timeouts.
  - Mock the network layer (nock/MSW) to intercept the queryFn's request and provide test data.
- Utilize jest setups and teardown for repeating logic [Jest Docs](https://jestjs.io/docs/setup-teardown)
- Utilize if needed existing helper function if exist for repetitive logic in multiple tests

---

## **Unit Tests → test the _smallest piece_ of behavior**

Use unit tests for **logic**, not flows.

Test in **unit**:

- Pure logic (computed values, derived state)
- Small UI behaviors in a single component
  (button enables/disables, text shows based on props)
- Error mapping / selection logic
  (deciding which error message to show)
- API calls _only if the component making the request is isolated_
  (like a hook or a small UI widget)

Don't test in unit:

- Anything requiring multiple components
- Routing, context, or global stores
- Full API flows (loading → success → result)

---

## **Integration Tests → test the _whole flow together_**

Use integration tests for **anything that spans multiple components or affects the app state.**

Test in **integration**:

- User flows (click → form → submit → result)
- Back navigation (history, routing)
- Routing behavior (URL params, redirects, protected routes)
- API flow logic (loading → success → error → UI update)
- Error UI is shown to the user
- State shared across components (Context/Redux/Zustand)
- Forms with multiple steps/components

Don't test in integration:

- Internal component details
- Styling
- Exact DOM structure

---

## React Native / Expo Additional Notes

When testing React Native / Expo, make sure to:

- Mock native modules: camera, location, async storage, secure store, permissions
- Wrap components using navigation: NavigationContainer for React Navigation
- Mock gestures/animations: Reanimated, Gesture Handler, Lottie
- Handle platform-specific code: mock Platform.OS for iOS vs Android branches
- React Query / network: still mock fetch/axios, but let React Query run normally
- Use helper functions / test utils for render wrappers (providers, query clients, stores)
- This ensures tests are deterministic, fast, and do not crash due to native APIs.
