---
name: shelve
description: Use when the user says to "shelve" or "shelf" a task, some, or all of them.
disable-model-invocation: false
allowed-tools: Bash(date:*), Bash(ls:*), Bash(mv:*), Bash(mkdir:*), Bash(git status:*), Write, Read
---

## Context

- Current shelf: !`ls .claude/shelf/ 2>/dev/null | sort || echo "(empty)"`
- Current time: !`date "+%Y-%m-%d %H:%M"`
- Git status: !`git status --short`

## Your task

Write each shelved task as a self-contained briefing a fresh session can pick up cold via `/task-shelf:shelf-pop`. Keep the shelf small (a handful at most); if a task won't be picked up soon, point the user at their issue tracker instead.

**One decision drives everything: is the current task shelved too, or kept live?** Infer from phrasing; ask if unclear.

- **Kept live** ("shelve the rest"): only the *other* tasks are written; finish the current one here, then pop the next.
- **Shelved too** ("shelve everything", "start fresh"): the current task is written alongside the rest; end by having the user `/clear` and pop.

Shelving the current task too needs a clean tree — underway work can't become a cold-start briefing. If it's shelved too and `git status` is dirty, STOP:

> "You have uncommitted work, so the current task is already underway — it can't be shelved as a fresh task. Finish and commit it (then pop the rest), or discard the changes to re-approach it from scratch. Don't commit partial work just to clear the tree; that splits one task across two commits."

(Keeping the current task live is fine with a dirty tree — you'll commit before popping.)

**Step 1 — Identify.** The current task and every remaining task. Decide (per above) whether the current one is written.

**Step 2 — Order.** Propose an order with a one-line rationale each; wait for confirmation before writing anything. Order by priority, but every task must come after the tasks it builds on (anywhere earlier, not necessarily adjacent), so prerequisites are done before it's popped.

**Step 3 — Number.** New tasks push to the **front**, numbered from `01`. If the shelf already has numbered files, bump them: renumber existing files sequentially from `1 + N` (N = new task count), renaming highest-first so nothing is overwritten mid-sequence. E.g. inserting 3 with existing `[03, 04]` → `04`→`05`, `03`→`04`; new tasks take `01, 02, 03`.

**Step 4 — Write.** Create `.claude/shelf/` if needed. For each task write `.claude/shelf/{NN}-{slug}.md` (`NN` zero-padded, `slug` a 2–4 word kebab label). Each file is a complete briefing for a session with zero prior history:

```
# {Task title}

**Shelved**: {datetime}
**Requires**: {Earlier work this builds on — any prior task, possibly several. Name each and describe the capability it delivers, conceptually — not as a file or commit reference, since neither survives the whole lifecycle. Omit if none.}

## Summary
{1–2 sentences: what needs doing and why}

## Context
{Architectural decisions, constraints, code locations, why it was deferred, what was discussed}

## Task
{Concrete description of what to implement or change}
```

**Step 5 — Confirm.**
- Kept live: "Shelved N tasks (01–0N). Continue working on {current task}."
- Shelved too: "Shelved N tasks (01–0N). Run `/clear` then `/task-shelf:shelf-pop` to begin."
