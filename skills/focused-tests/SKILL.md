---
name: focused-tests
description: >-
    Test-writing guidance for focused, valuable coverage based on feature contracts and
    user-visible behavior instead of implementation details. Use when writing or
    reviewing frontend, backend, unit, integration, or end-to-end tests.
---

# Core principles:

- Tests should be focused on the feature contract and user-visible behavior, not implementation details.
- Write tests in a way that enables developers to be able to refactor the software without needing to change the tests.
- Focus on the feature behavior and business outcomes of the tested component, class or whatever u are testing.
- Cover the main success path, unhappy paths, realistic failures, important edge cases, and meaningful boundary conditions.
- Make user-visible error states explicit when they affect behavior, but do not test exact copy.

# Never do this

- Don't couple tests to implementation its like everything u change in code u have to change the tests to perfectly match it.
- Don't test exact UI text, button labels, CSS, DOM structure, or server response wording.
