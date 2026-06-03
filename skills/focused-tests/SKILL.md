---
name: focused-tests
description: >-
    Skill for writing only the valuable test cases for a feature, not an exhaustive checklist of every observable detail. Can be used for both frontend and backend tests, 
    and for any type of test (unit, integration, end-to-end, etc.) as long as the focus is on the feature contract and user-visible behavior rather than implementation details.
    Trigger whenever any test is being written, edited, fixed, or refactored — including fixing tests that broke after a code change, adding assertions
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
