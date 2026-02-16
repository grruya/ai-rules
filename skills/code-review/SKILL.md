---
name: review
description: Multi-step code review
---

# Review: $ARGUMENTS

## Step 1: Discovery
Use feature-file-discovery subagent with prompt: "Find all files related to: $ARGUMENTS"
Store the file list.

## Step 2: Code Execution Flow Mapping  
Use execution-flow subagent with prompt: "Map the flow for these files: [file list from Step 1]"
Store the flow path.

## Step 3: Parallel Review
Launch these subagents in parallel, passing the file list and flow path:
- security-reviewer
- logic-reviewer
- performance-reviewer

## Step 4: Consolidation
Use deduplicate-issues subagent to deduplicate and sort all issues.

## Output Format
```
📁 Files Reviewed (5):
  - src/auth/LoginController.php
  - src/middleware/Authenticate.php
  - src/services/TokenService.php
  - src/models/User.php
  - routes/api.php

📊 Flow Path:
  HTTP Request → Authenticate middleware → LoginController@login 
  → TokenService::generate → User::findByEmail → Response

🔍 Found 3 issues :

[CRITICAL] Missing rate limiting on login endpoint
  src/auth/LoginController.php:23
  → Brute force attacks possible
  Fix: Add throttle middleware to route

[HIGH] Token not invalidated on logout
  src/services/TokenService.php:45
  → Old tokens remain valid indefinitely
  Fix: Implement token blacklist or short expiry

[MEDIUM] Missing null check on user lookup
  src/auth/LoginController.php:28
  → Could throw exception if user doesn't exist
  Fix: Add null check before proceeding
```