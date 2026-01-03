---
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*), Bash(git push:*)
argument-hint: [message]
description: Commit all changes and push to current branch
---

# Commit and Push

Commit all staged changes with the provided message and push to the current branch.

## Context

Current branch: !`git branch --show-current`

Current status: !`git status --short`

Recent commits: !`git log --oneline -3`

## Your task

1. Stage all changes: `git add -A`
2. Commit with message: "$ARGUMENTS"
3. Push to the current branch: `git push origin <current-branch>`

Always use the current branch (not main), and never force push.
