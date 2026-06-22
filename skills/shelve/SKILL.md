---
name: shelve
description: Shelve remaining tasks onto the task shelf (stack push)
disable-model-invocation: false
allowed-tools: Bash(date:*), Bash(ls:*), Bash(mv:*), Bash(mkdir:*), Bash(git status:*), Write, Read
---

## Context

- Current shelf: !`ls .claude/shelf/ 2>/dev/null | sort || echo "(empty)"`
- Current time: !`date "+%Y-%m-%d %H:%M"`
- Git status: !`git status --short`

## Your task

The user wants to shelve tasks from the current conversation to focus on one thing at a time. Shelved tasks become self-contained briefing files that a fresh session picks up cold via `/task-shelf:shelf-pop`.

The whole skill turns on **one decision: does the task being worked on right now get shelved too, or stay live in this session?**

- **Stay live** ("shelve the rest", "shelf the remaining"): the current task keeps going here; only the *other* tasks go to the shelf. You finish the current one in this session, then pop the next.
- **Shelve it too** ("shelve everything", "start fresh", "planning shelve"): nothing stays live; the current task is written to the shelf alongside the rest, and you end by clearing the session and popping the first one clean.

Infer which from the user's phrasing. If it's genuinely unclear, ask.

**Shelving the current task too requires a clean working tree.** A dirty tree means the current task is already underway, and underway work can't become a cold-start briefing. So if the user wants to shelve the current task and `git status` shows uncommitted changes, STOP:

> "You have uncommitted work, so the current task is already underway — it can't be shelved as a fresh task. Finish it and commit it (then pop the rest), or discard the changes if you want to re-approach it from scratch. Don't commit partial work just to clear the tree; that splits one task across two commits."

(The "stay live" path is unaffected — a dirty tree is expected there, since you'll finish and commit the current task before popping.)

**Step 1: Identify the tasks**

From the conversation, identify the task being worked on right now and every remaining task to shelve. Decide (per above) whether the current task is included in what gets written.

**Step 2: Propose ordering**

Present the tasks to be shelved in your proposed priority order with a one-line rationale for each. Wait for the user to confirm or reorder before creating any files.

**Step 3: Number the tasks and make room**

New tasks are a stack push — they go at the **front**, numbered sequentially starting at `01`.

If `.claude/shelf/` already has numbered files, bump them to make room: sort existing files by number, then renumber them sequentially starting at `1 + N`, where N is the count of new tasks. Rename in reverse sorted order (highest number first) so you never overwrite a file mid-sequence. Example: inserting 3 tasks with existing `[03, 04]` → rename `04` → `05`, then `03` → `04`; the new tasks take `01, 02, 03`.

**Step 4: Create the briefing files**

Create `.claude/shelf/` if it doesn't exist. For each task in the confirmed order, write `.claude/shelf/{NN}-{slug}.md`:
- `NN` is zero-padded (01, 02…)
- `slug` is a short kebab-case label derived from the task title (2–4 words)

Each file must be a **complete briefing for a fresh Claude session with zero prior conversation history**. Write with enough context that someone starting cold can immediately dive in:

```
# {Task title}

**Shelved**: {datetime}
**Requires**: {What the preceding task delivers that this task builds on — omit if this is the first task or has no dependencies}

## Summary
{1–2 sentences: what needs to be done and why}

## Context
{Full context: relevant architectural decisions, constraints, code locations, why this was deferred, what was discussed}

## Task
{Concrete description of what to implement or change}
```

**Step 5: Confirm**

- Current task stayed live: "Shelved N tasks (01–0N). Continue working on {current task}."
- Current task shelved too: "Shelved N tasks (01–0N). Run `/clear` then `/task-shelf:shelf-pop` to begin."
