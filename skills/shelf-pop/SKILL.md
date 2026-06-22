---
name: shelf-pop
description: Pop the next task off the task shelf and load its full context to start working on it. Requires a clean working tree (one task, one commit). Use when the user says "pop the shelf", "next shelved task", or "pick up the next task".
disable-model-invocation: false
allowed-tools: Bash(git status:*), Bash(git log:*), Bash(git show:*), Bash(ls:*), Bash(rm:*), Read
---

## Context

- Git status: !`git status --short`
- Shelf contents: !`ls .claude/shelf/ 2>/dev/null | sort | grep -E '^[0-9]' || echo "(empty)"`

## Your task

The user has finished the current task and wants to start the next shelved one.

**Step 1 — Check git.** If `git status --short` is dirty, STOP: "Uncommitted changes detected. Commit everything before popping the shelf — one task, one commit."

**Step 2 — Find.** Take the lexicographically smallest numbered file in `.claude/shelf/` (`01-…` before `02-…`). If none: "The shelf is empty — you're all done!" and stop.

**Step 3 — Load.** Read the file fully, then `rm` it.

**Step 4 — Context.** Read the file's `**Shelved**: {datetime}`, run `git log --oneline -10 --after="{datetime}"`, and `git show <hash>` any commit whose message looks related to this task.

**Step 5 — Present.** Always explain the task to the user: what it is, when it was shelved, and which work since then (from the related commits) bears on it — skip unrelated ones. Suggest how to start. Finish with: "Run `/rename` to name this session after the current task."
