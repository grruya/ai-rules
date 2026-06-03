---
name: review-feature
description: >-
    Command-style skill for running the multi-agent feature review pipeline. Use
    only when the user explicitly invokes `/review-feature`, tags this skill, or
    asks for it by name.
---

# Full code review (multi-agent)

- Run the **change review pipeline** by orchestrating the custom Codex agents defined in `.codex/agents/`. Keep the parent agent thin: resolve the target, spawn the agents, wait, and format the result. Do not do the reviewers' work in the parent thread.
- Keep all handoff packets compact. If a subagent needs code, let it inspect the repository itself.
- Call `spawn_agent` for the four reviewers before calling `wait_agent`; do not run reviewers serially.

## Step 1 — Map the full feature execution flow

Start from the feature or subsystem the user asked to review. Use that as the entry point to understand how the full feature works.

Your goal in this step is to identify the **minimal complete feature scope** and how the feature behaves end to end.

Scope should include:

- directly relevant or explicitly requested files
- entry points for the feature
- immediate callers and callees that materially affect behavior
- related validation, authorization, configuration, schema, jobs, events, policies, and other files that change runtime behavior
- side-effect paths such as notifications, payments, persistence, cache, queues, and integrations

Do not expand scope into unrelated parts of the repo. Expand only until the feature behavior is understandable end to end.

Spawn the **`execution-flow`** agent with the initial feature description or starting files for that feature, then wait for its result.

**Prompt:**

```
Map the full execution flow for this feature:
{user-requested feature or starting file list}

Your job:
- identify the full feature scope, even if some files were not explicitly requested
- include all related files needed to understand how the feature behaves end to end
- trace the real execution flow through the system

Also identify:
- entry points
- trust boundaries
- authorization and validation checkpoints
- state mutations
- side effects
- error and exception paths

Output a compact packet only:
- `feature_files`: max 20 paths
- `execution_flow`: max 14 bullets
- `review_notes`: max 10 bullets with only facts reviewers need
```

**Store for later steps:**

- the expanded feature file list, capped at 20 paths unless the feature truly requires more
- the compact execution flow

## Step 2 — Parallel reviews using feature flow

Spawn these four agents **in parallel**. Call `spawn_agent` four times first, then call `wait_agent` for all four results:

- `security-reviewer`
- `logic-reviewer`
- `performance-reviewer`

Each agent already encodes what to look for. Pass all of the following to each one:

- the expanded feature file list from Step 1
- the execution flow from Step 1

**Prompt template (each reviewer):**

```
Review this feature:

Feature files:
{expanded feature file list from Step 1}

Execution flow:
{execution flow from Step 1}

Rules:
- inspect repository files directly instead of asking the parent for full file contents
- return `No findings` if you do not find a concrete issue
- max 5 findings

Use your agent rubric. Report only concrete, reachable findings tied to this flow.
```

**Collect all issues** from the four agent results.

## Step 3 — Consolidate

Spawn **`deduplicate-issues`** with every issue from Step 2, then wait for the result.

**Prompt:**

```
Deduplicate and organize these issues from multiple reviewers. If all inputs say `No findings`, return exactly `No findings`.
{all issues from Step 2}

Preserve each issue’s source agent id(s). When merging duplicates reported by different agents, keep all contributing agent names on that issue.
Keep the review finding format.
```
