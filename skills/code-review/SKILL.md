---
name: review
description: >-
  Multi-step code review workflow. Activates when the user requests a code review,
  security audit, or analysis of a feature/flow. Use this skill to perform
  comprehensive code reviews that discover files, map execution flows, and
  identify security, logic, and performance issues.
---

# Code Review Workflow

This skill orchestrates a comprehensive code review by delegating to specialized subagents. Each subagent's behavior is defined in `.cursor/agents/` - this skill focuses on orchestration and data flow.

## Workflow Steps

### Step 1: Discovery
Delegate to the `feature-file-discovery` subagent with the user's feature/flow description.

**Prompt:**
```
Find all files related to: {user's feature description}
```

**Store the file list** returned by the subagent for use in subsequent steps.

### Step 2: Code Execution Flow Mapping
Delegate to the `execution-flow` subagent with the file list from Step 1.

**Prompt:**
```
Map the execution flow for these files:
{file list from Step 1}
```

**Store the flow path** returned by the subagent for use in the review steps.

### Step 3: Parallel Review
Launch these three subagents **in parallel** (using multiple Task tool calls in a single message):
- `security-reviewer`
- `logic-reviewer`
- `performance-reviewer`

For each subagent, provide the file list and flow path. The subagents already know what to check for based on their definitions.

**Prompt template for each reviewer:**
```
Review these files:
{file list from Step 1}

Execution flow:
{flow path from Step 2}
```

**Collect all results** from the three parallel reviews.

### Step 4: Consolidation
Delegate to the `deduplicate-issues` subagent with all issues from Step 3.

**Prompt:**
```
Deduplicate and organize these issues from multiple reviewers:
{all issues from Step 3}
```

## Output Format

After completing all steps, present the results in this format:

```
📁 Files Reviewed ({count}):
  - {file path 1}
  - {file path 2}
  - ...

📊 Flow Path:
  {execution flow description from Step 2}

🔍 Found {count} issues:

[CRITICAL] {issue title}
  {file path}:{line number}
  → {description of the problem}
  Fix: {suggested fix}

[HIGH] {issue title}
  {file path}:{line number}
  → {description of the problem}
  Fix: {suggested fix}

[MEDIUM] {issue title}
  {file path}:{line number}
  → {description of the problem}
  Fix: {suggested fix}
```

## Important Notes

- **Subagent Definitions**: Each subagent's behavior is already defined in `.cursor/agents/`. You don't need to repeat their instructions - just invoke them with the necessary data.
- **Context Isolation**: Each subagent starts with a clean context. Include all necessary information (file lists, flow paths) in the prompt you send to each subagent.
- **Parallel Execution**: Launch the three review subagents in Step 3 simultaneously using multiple Task tool calls in a single message for efficiency.
- **Data Passing**: Since subagents don't share context, explicitly include all relevant data (file lists, flow paths) in each subagent's prompt.