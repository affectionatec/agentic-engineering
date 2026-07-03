---
name: status-tracker
description: Use at the start of every coding session (read docs/status.md before any other action), at the end of every session (append handoff log entry), or when checking current progress, last-done work, in-flight tasks, or what's blocked. Triggers on session start/end, "where are we", "pick up where we left off", "handoff", "what was done last session", "resume", or recovering after context loss / new agent session.
---

# Status Tracker — Project Status & Progress

> **Core constraint:** This file is the Agent's memory. Read it first, update it last. Every session.

## Quick Reference

| | |
|---|---|
| **Use when** | Start of every session (read first), end of every session (update last) |
| **Skip when** | Never — every session reads `docs/status.md` before any other action |
| **Output** | `docs/status.md` — live, with append-only handoff log |
| **Sequence** | Created when IMPL PLAN starts; updated every session forever |
| **Iron rule** | Handoff log is append-only, newest first. Never edit past entries. |
| **Sibling skills** | [[project-kickoff-prd]] · [[technical-specification]] · [[architecture-decision-record]] · [[implementation-plan]] · [[independent-verification]] · [[git-workflow]] |

## Role

You are a disciplined engineering lead responsible for maintaining the project's living status tracker. This document is the bridge between sessions — it is what makes agentic coding sustainable across days, weeks, and months.

### When to Create

Create `docs/status.md` as soon as the first implementation task begins. It lives alongside the implementation plan and is referenced from `AGENTS.md` (the documentation chain table) — every tool's pointer file leads there.

### Session Protocol

#### At the Start of Every Session

1. **Read `docs/status.md` before doing anything else.** Understand the current phase, what was done last, and what was queued for this session.
2. **Check the In-Flight Checkpoint.** If it is not `none`, the previous session ended abnormally (crash, context exhaustion, interruption) — the checkpoint is your recovery point. Resume from its "next step", then note the recovery in this session's handoff entry.
3. **Read the latest handoff log entry.** This is your briefing. Follow its "next session should…" instruction unless the user redirects.

#### During the Session — Checkpoint Cadence

Long sessions must not hold progress only in the context window. Write intermediate state at task granularity — not every step (wasteful), not only at session end (catastrophic on failure):

1. **After every completed task:** update the In-Flight Checkpoint, flip the module table entry (🟡 → 🔍 or ✅), and refresh the header block.
2. **Before any long or risky operation** (large refactor, migration, anything that could exhaust context): update the In-Flight Checkpoint first.
3. A crash should cost you at most one task of progress, never the session.

#### At the End of Every Session

1. **Update the module progress table.** Move status emojis, update notes and next actions.
2. **Update the header block.** Current phase, next up, code status (test count, lint, build).
3. **Append a handoff log entry.** This is the most important update. Be specific and thorough.
4. **Reset the In-Flight Checkpoint to `none`.** A clean checkpoint is the signal that the session ended properly and the handoff entry is authoritative.
  
### Document Structure

The file must contain exactly these sections, in this order:

> Save as `docs/status.md`

````markdown
# Project Status & Progress

> **Single source of truth for "where we are."** Read at the start of every session; update at the end. The stable build plan is implementation-plan.md; **this file tracks live progress** against it.

- **Current phase:** ...
- **Next up:** ...
- **Code status:** ...

## In-Flight Checkpoint

> Live scratch state for the current session only. Overwritten freely while working; reset to `none` at session end. **If this is not `none` at session start, the previous session crashed — recover from here.**

none

<!-- When active:
- **Active task:** M2-T3 — [title] (🟡)
- **Done so far:** [completed steps, files changed, tests added]
- **Next step:** [the literal next action]
- **Scratch:** [branch name, failing test, half-made decision — whatever resuming needs]
-->

## Module Progress

Plan & contracts: implementation-plan.md. Legend: ⬜ not started · 🟡 in progress · 🔍 built, awaiting verification · ✅ verified done

| Module | Status | Notes / next action |
| ------ | :----: | ------------------- |
| ...    |  ...   | ...                 |

## Decisions

Locked in adr/. **Do not relitigate** — raise changes with the user.

## Open Items (non-blocking)

- ...

## Session Handoff Log

Newest first.

- **YYYY-MM-DD** — ...
````

### Section Rules

#### Header Block (Current Phase / Next Up / Code Status)

- **Current phase** — The active milestone, its short description, and a status indicator (⬜ / 🟡 / 🔍 / ✅). If a PR is open, say so — one task, one branch, one PR (→ [[git-workflow]]).
- **Next up** — The next milestone and its precondition (e.g., "once M3 merges, start M4").
- **Code status** — Cumulative merged state + current working state. Always include: merged milestone range, test count (exact number), lint status (clean or not), and any caveats (e.g., "⚠️ live API needs keys").

#### In-Flight Checkpoint

- **Live state, not history.** This is the only section that gets overwritten instead of appended — it exists so a crash mid-session loses at most one task, not the session.
- Update it after every completed task and before any long or risky operation.
- At session end, after the handoff entry is written, reset it to `none`. The handoff log is the authoritative record; a non-`none` checkpoint at session start means abnormal termination.
- Keep it resumable: a fresh agent reading only this section should know exactly what to do next.

#### Module Progress Table

- One row per module/milestone from the implementation plan.
- **Status** column uses only: `⬜` (not started), `🟡` (in progress), `🔍` (built, awaiting verification), `✅` (verified done/merged).
- **`✅` requires a verifier verdict, not a producer claim.** Work the producer believes is finished sits at `🔍` until the independent verifier returns PASS (→ [[independent-verification]]); the notes column references the verdict (date + task ID in `docs/verification-log.md`).
- **Notes** column must be actionable — not "working on it" but "built & verified locally (138 tests); PR open" or "blocked on M5 merge."
- Keep notes concise but specific. Include test counts, key components built, and the literal next action.

#### Decisions

- Point to the ADR directory. One line.
- Enforce the rule: decisions are locked. Do not re-open them without the user's explicit approval.

#### Open Items

- Non-blocking issues, known limitations, deferred work.
- Each item should explain *why* it's non-blocking and *when* it would become blocking.
- Remove items once resolved. This is not an append-only log.

#### Session Handoff Log (Critical)

This is the most important section. Each entry must contain:

1. **Date** — Bold, ISO format.
2. **What was built** — Module name, file paths created/modified, key components and their responsibilities.
3. **Key design details** — Architecture choices made, patterns used, how things connect to existing code. Enough detail that a reader understands *how* it was built, not just *that* it was built.
4. **What was verified** — Exact test count (total and new), lint status, specific test scenarios that matter (e.g., "PIT no-future-leak," "cache hit skips network").
5. **Caveats** — Anything that works locally but needs a special environment, API keys, network access, etc. Mark with ⚠️.
6. **What the next session should do** — Explicit instruction. Not "continue working" but "once M7 merges, start M8 (LLM infra — multi-provider client, caching, R2 copilot)."

**Handoff log entries are append-only and newest-first.** Never edit a past entry. If a past entry was wrong, note the correction in the current entry.

### Formatting Rules

1. **Be exact with numbers.** "232/232 pytest" not "all tests pass." "14 new tests" not "added tests."
2. **Name the files.** `src/quant/intelligence/doc_store.py` not "the doc store module."
3. **Name the components.** `DocStore PIT Parquet` not "the storage layer."
4. **Link to the plan.** The module table should reference the implementation plan sections.
5. **Use consistent emoji.** ⬜ 🟡 🔍 ✅ for status. ⚠️ for caveats. No others.
6. **Keep the header block scannable.** Someone should understand project state in 5 seconds from the top three lines.

### Anti-Patterns (Do Not)

1. **Do not skip the handoff log update.** Even if the session was short or exploratory, log what happened.
2. **Do not write vague handoff entries.** "Made progress on M5" is useless. Say what was built, what was tested, what's next.
3. **Do not update the module table without updating the header.** They must stay in sync.
4. **Do not log decisions here.** Decisions go in ADRs. The status file only *references* them.
5. **Do not let open items accumulate silently.** Review and prune every few sessions.
6. **Do not mix session logs.** One entry per session. If a session spans two milestones, log both in the same entry.
7. **Do not mark ✅ on a producer claim.** ✅ means the independent verifier returned PASS. Until then the honest state is 🔍.
8. **Do not let the In-Flight Checkpoint go stale.** A checkpoint pointing at last week's task is worse than `none` — the next agent will resume the wrong work. Update it at task granularity, reset it at session end.


