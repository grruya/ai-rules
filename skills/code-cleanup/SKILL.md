---
name: code-cleanup
description: >-
    Command-style skill for cleaning up maintainability issues in code. Use only
    when the user explicitly invokes `/code-cleanup`, tags this skill, or
    asks for it by name.
---

# Code Cleanup

Manual activation only: use this skill only when the user explicitly invokes
`/code-cleanup`, tags this skill, or asks for this cleanup command by
name.

Clean up code so it follows the current project patterns, keeps files and
functions focused, uses honest names, returns meaningful results, and avoids
unnecessary defensive checks or type checks.

Assume the target code may be AI-generated. Be especially alert for AI-style
overengineering:

- redundant null checks
- repeated type checks
- impossible defensive branches
- wrapper abstractions with no purpose
- verbose guards around values already guaranteed by validation, framework
  contracts, type declarations, or nearby code

## Workflow

1. Resolve the cleanup target from the user's request.
   This may be a file, directory, diff, branch, staged changes, unstaged changes,
   or a described feature area.

2. Make focused cleanup edits for concrete maintainability problems:
    - project pattern mismatches where nearby code clearly uses a different shape
    - poor separation of concerns, such as a file or class named for one
      responsibility also performing unrelated side effects
    - misleading names, such as a function named like a predicate returning
      `void`, or a command-style method returning hidden state
    - unclear return semantics, such as `checkPlaceAvailability` returning
      nothing instead of a boolean, available place, or `null`
    - unnecessary defensive checks, type checks, or fallback branches where the
      code already guarantees the type, presence, or state
    - duplicated ceremony or helper layers that do not simplify the feature
    - dead code or code that is used but it does not solve the problem so it can be removed
    - fallbacks without real meaning

3. Verify the result.
   Run the most relevant formatter, typecheck, tests, or focused command
   available in the repository. If verification cannot be run, explain why.
