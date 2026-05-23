---
name: prepare-pr-branch
description: >-
    Command-style skill for preparing a child branch from the current branch,
    committing dirty changes, and writing a PR summary back into the original
    branch. Use only when the user explicitly invokes `/prepare-pr-branch`,
    tags this skill, or asks for it by name.
---

# Prepare PR Branch

Manual activation only: use this skill only when the user explicitly invokes
`/prepare-pr-branch`, tags this skill, or asks for prepare-pr-branch by name.

## Goal

Create a local branch from the branch the user is currently on, commit any
uncommitted changes, and generate a PR summary for merging the new branch back
into the original branch.

## Workflow

1. Record the current branch with `git branch --show-current`.
2. Inspect status with `git status --short`.
3. Create and switch to a new branch from the current `HEAD`.
   - If the user gives a branch name, use it.
   - Otherwise choose a short local branch name with the `codex/` prefix.
4. If there are uncommitted changes:
   - Stage the relevant changes.
   - Read the staged diff.
   - Use the message rules from `skills/caveman-commit/SKILL.md`.
   - Commit the staged changes with that message.
5. Generate a PR summary for merging the new branch into the original branch.
   - Use `git diff <original-branch>...HEAD` and commit history as needed.
   - Use the structure and tone from `skills/caveman-pr/SKILL.md`.
   - Make the base branch explicit.
6. Report the created branch, commit status, and PR summary.

## Rules

- Do not run this skill unless it was explicitly called by name, tag, or slash
  command.
- Do not push or create a remote pull request unless the user asks.
- Do not overwrite an existing branch name; choose another name or ask when the
  requested name already exists.
- If the working tree has unrelated changes that should not be included, stop
  and ask what to stage.
- If there are no uncommitted changes, still create the branch and generate the
  PR summary from the branch diff/history.
