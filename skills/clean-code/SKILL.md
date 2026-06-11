---
name: clean-code
description: >-
    Apply concise, language-agnostic clean-code standards only when writing,
    editing, refactoring, or generating code. Never use this skill outside of writing, editing or giving code snippets.
---

# Clean Code Standards

Use this skill only while producing code changes or code snippets.

**Note:** This file uses php syntax but its language agnostic so this rules apply for every language not just php

## Rules

- **Early Returns:** Avoid nested `if` statements. Use guard clauses (e.g., `if (!$condition) return;`) to keep the "happy path" at the lowest indentation level.
- **Simplicity:** Favor simplicity over "clever" code. Avoid recursion, deep callbacks, and complex abstractions unless strictly necessary.
- **Method Limits:** Methods should have a maximum of **4 parameters**.
- **Scope:** Keep variables as close as possible to the block where they are consumed.

## Writing Style

- Write code like reading instructions: a linear sequence of clear steps where each line explains what happens next, avoiding hidden logic, deep nesting, or unclear jumps in execution.
- Never place complex logic inside if parentheses. Extract complex conditions into descriptive boolean variables before the if statement.
- Never add comments to my code no matter what

## Example

```php
enum Clothing: string
{
    case Shirt = 'shirt';
    case Jacket = 'jacket';
    case Hoodie = 'hoodie';
}

enum WeatherCondition: string
{
    case Sunny = 'sunny';
    case Rain = 'rain';
    case Snow = 'snow';
}

/**
 * @return array{temperature: int, condition: ?WeatherCondition}
 */
function getForecastForToday(): array

function whatToWearToday(): Clothing
{
    try {
      $weather = getForecastForToday();
    } catch (WeatherApiException $e) {
      // Handle error
    }

    if ($weather->temperature > 30) return Clothing::Shirt;
    if ($weather->temperature < 10) return Clothing::Jacket;

    return Clothing::Hoodie;
}
```
