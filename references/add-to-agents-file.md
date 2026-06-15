# Code style preferences

- Avoid naming files or functions containing this words "normalize, build, resolve" be explicit with naming and what it does
- Name function so its predictable what they return, for example: applyClothing should be void, whatToWearToday should return enum or string.
- Avoid misleading names, such as a function named like a predicate returning `void`, or a command-style method returning hidden state

# Testing

- Tests should mirror the project directory structure
