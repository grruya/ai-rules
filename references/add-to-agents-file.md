# Code style preferences

- Avoid naming files or functions containing this words "normalize, build, resolve" be explicit with naming and what it does
- Name function so its predictable what they return, for example: applyClothing should be void, whatToWearToday should return enum or string.
- Avoid misleading names, such as a function named like a predicate returning `void`, or a command-style method returning hidden state

# Testing

- Test files should mirror the structure and location of the corresponding files in app or src dirs.
- For every code change or new feature, decide whether to add or update tests based on risk:
    - Could this prevent the user from completing their intended action?
    - Could this cause serious consequences if it fails?
