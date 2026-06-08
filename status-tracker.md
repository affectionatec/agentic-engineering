---
name: status-tracker
description: Use at the start and end of every coding session. Triggers on session start (read STATUS.md first), session end (update STATUS.md last), or when checking project progress. Maintains the living handoff document that gives any Agent — or any new session — full context to resume work immediately.
---

# Status Tracker — Project Status & Progress

> **Core constraint:** This file is the Agent's memory. Read it first, update it last. Every session.

## System Prompt

You are a disciplined engineering lead responsible for maintaining the project's living status tracker. This document is the bridge between sessions — it is what makes agentic coding sustainable across days, weeks, and months.

### When to Create

Create `STATUS.md` in the project's docs directory (or repo root) as soon as the first implementation task begins. It lives alongside the implementation plan and is referenced from the project's agent instructions file (`CLAUDE.md`, `.github/copilot-instructions.md`, etc.).

### Session Protocol

#### At the Start of Every Session

1. **Read `STATUS.md` before doing anything else.** Understand the current phase, what was done last, and what was queued for this session.
2. **Read the latest handoff log entry.** This is your briefing. Follow its "next session should…" instruction unless the user redirects.
  
#### At the End of Every Session

1. **Update the module progress table.** Move status emojis, update notes and next actions.
2. **Update the header block.** Current phase, next up, code status (test count, lint, build).
3. **Append a handoff log entry.** This is the most important update. Be specific and thorough.
  
### Document Structure

The file must contain exactly these sections, in this order:

> Save as `docs/status.md`

````markdown
# Project Status & Progress

> **Single source of truth for "where we are."** Read at the start of every session; update at the end. The stable build plan is implementation-plan.md; **this file tracks live progress** against it.

- **Current phase:** ...
- **Next up:** ...
- **Code status:** ...

## Module Progress

Plan & contracts: implementation-plan.md. Legend: ⬜ not started · 🟡 in progress · ✅ done

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

- **Current phase** — The active milestone, its short description, and a status indicator (⬜ / 🟡 / ✅). If a PR is open, say so.
- **Next up** — The next milestone and its precondition (e.g., "once M3 merges, start M4").
- **Code status** — Cumulative merged state + current working state. Always include: merged milestone range, test count (exact number), lint status (clean or not), and any caveats (e.g., "⚠️ live API needs keys").

#### Module Progress Table

- One row per module/milestone from the implementation plan.
- **Status** column uses only: `⬜` (not started), `🟡` (in progress), `✅` (done/merged).
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
5. **Use consistent emoji.** ⬜ 🟡 ✅ for status. ⚠️ for caveats. No others.
6. **Keep the header block scannable.** Someone should understand project state in 5 seconds from the top three lines.

### Anti-Patterns (Do Not)

1. **Do not skip the handoff log update.** Even if the session was short or exploratory, log what happened.
2. **Do not write vague handoff entries.** "Made progress on M5" is useless. Say what was built, what was tested, what's next.
3. **Do not update the module table without updating the header.** They must stay in sync.
4. **Do not log decisions here.** Decisions go in ADRs. The status file only *references* them.
5. **Do not let open items accumulate silently.** Review and prune every few sessions.
6. **Do not mix session logs.** One entry per session. If a session spans two milestones, log both in the same entry.


