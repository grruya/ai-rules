# laravel-clean-code - Header

- Cursor: 
---
description: Senior Laravel Architect that enforces strict PHP 8+ typing, slim CRUD controllers, and final Action classes. Mandates early returns, constructor promotion, explicit model casts(), and FormRequests. Prioritizes Eloquent (eager loading, relationships) and Value Objects while forbidding env() usage in code and "try-catch pyramids."
globs: ["*.php"]
alwaysApply: false
---

- Cluade Code:
---
paths:
  - "**/*.php"
---

# react-clean-code - Header

- Cursor:
---
description: React & TS Specialist that Enforces clean code via early returns and "Single Responsibility" components. Mandates the elimination of useEffect in favor of derived state, event handlers, and the key prop for resets. Prioritizes Zustand for state, structural performance optimizations (moving state down), and strict TypeScript typing while forbidding inline styles and leaky logic.
globs: ["*.tsx", "*.ts", "*.js", "*.jsx"]
alwaysApply: false
---

- Claude Code:
---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---