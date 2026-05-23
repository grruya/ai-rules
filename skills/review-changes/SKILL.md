---
name: review-changes
description: >-
    Command-style skill for running the multi-agent change review pipeline. Use
    only when the user explicitly invokes `/review-changes`, tags this skill, or
    asks for it by name.
---

# Review changes (multi-agent)

Manual activation only: use this skill only when the user explicitly invokes
`/review-changes`, tags this skill, or asks for this review command by name.

Run the **change review pipeline** by orchestrating the custom Codex agents defined in `.codex/agents/`. Keep the parent agent thin: resolve the target, spawn the agents, wait, and format the result. Do not do the reviewers' work in the parent thread.

Token budget rules:

- Do not paste whole file contents into subagent prompts.
- Do not paste the full raw diff unless the review target is a tiny patch. Prefer changed file paths plus changed hunks or a short diff summary.
- Keep all handoff packets compact. If a subagent needs code, let it inspect the repository itself.
- Call `spawn_agent` for the four reviewers before calling `wait_agent`; do not run reviewers serially.

This pipeline is for **change-level review only**.

The reviewers should understand the wider execution flow, but they should review and report findings only for the **changed files** in the requested change set.

## Step 1 — Resolve the review target

Determine exactly what the user wants reviewed.

The user may describe the review target in any reasonable way, for example:

- a PR diff
- the last commit
- a specific earlier commit such as the second commit in a branch
- staged changes
- unstaged changes
- an explicit diff range
- a list of files

Resolve and store a compact review packet:

- the user-requested review target
- the exact changed or requested file list
- changed hunks or a short change summary
- base/head refs when available

## Step 2 — Map execution flow for context

Spawn the **`execution-flow`** agent using the changed or requested files from Step 1 as the starting point, then wait for its result.

Your goal in this step is **not** to widen the review target. Your goal is to build enough context so reviewers understand how the changed files behave inside the larger system.

Ask the subagent for a compact context packet, not a full essay:

- the execution flow touching the changed files
- entry points
- immediate callers and callees that materially affect the changed behavior
- validation, authorization, configuration, schema, jobs, events, policies, and side effects relevant to the changed behavior
- trust boundaries
- state mutations
- error and exception paths

Do not turn the whole related feature into the review target. Expand only enough to understand the changed behavior end to end.

**Prompt:**

```text
Map the execution flow needed to understand this review target:
{changed or requested files from Step 1}

Your job:
- trace the real execution flow needed to understand these changed files
- include only the supporting files and paths needed for context
- do not redefine the review target beyond the changed or requested files

Also identify:
- entry points
- trust boundaries
- authorization and validation checkpoints
- state mutations
- side effects
- error and exception paths

Output a compact packet only:
- `supporting_context_files`: max 12 paths
- `execution_flow`: max 12 bullets
- `review_notes`: max 8 bullets with only facts reviewers need
```

**Store for later steps:**

- the changed or requested file list
- the compact execution flow
- the supporting context files from the flow, capped at 12 paths

## Step 3 — Parallel reviews using change target and execution flow

Spawn these four agents **in parallel**. Call `spawn_agent` four times first, then call `wait_agent` for all four results:

- `security-reviewer`
- `logic-reviewer`
- `performance-reviewer`
- `code-quality-reviewer`

Each agent already encodes what to look for. Pass all of the following to each one:

- the changed or requested file list from Step 1
- the changed hunks or short change summary from Step 1
- the compact execution flow from Step 2
- the supporting context files from Step 2, capped at 12 paths

**Prompt template (each reviewer):**

```text
Review this change set.

Changed or requested files:
{changed or requested file list from Step 1}

Supporting context files:
{supporting context files from Step 2}

Changed hunks or summary:
{changed hunks or short change summary from Step 1}

Execution flow:
{execution flow from Step 2}

Rules:
- only review changed files, do not review files in execution flow use them just for context
- do not report pre-existing issues that exist only in unchanged files
- keep findings concrete, reachable, and tied to this change set
- inspect repository files directly instead of asking the parent for full file contents
- return `No findings` if you do not find a concrete issue

Use your agent rubric. Report only concrete, reachable findings tied to this change set.
```

**Collect all issues** from the four agent results.

## Step 4 — Consolidate

Spawn **`deduplicate-issues`** with every issue from Step 3, then wait for the result.

**Prompt:**

```text
Deduplicate and organize these issues from multiple reviewers. If all inputs say `No findings`, return exactly `No findings`.
{all issues from Step 3}

Preserve each issue’s source agent id(s). When merging duplicates reported by different agents, keep all contributing agent names on that issue.
Keep the review finding format.
```
