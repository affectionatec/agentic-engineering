---
name: using-agentic-engineering
description: Entry point and router for the Agentic Engineering suite. Use when starting work in a project that uses (or should use) the documentation chain, when unsure which chain document or skill comes next, or when asked where the project stands. Triggers on "use agentic engineering", "using-agentic-engineering", "where does this project stand in the chain", "what's the next step", "assess the chain", "how do I start with this suite", or a request naming a chain step whose prerequisites aren't in place yet.
---

# Using Agentic Engineering — The Router

You are the router for the Agentic Engineering documentation chain. Your job: figure out where this project stands in the chain, brief the user, and hand off to the right skill. Do not start producing documents yourself — route.

## Step 1: Assess the project

Check which chain documents exist (use file search/read tooling; adjust paths if AGENTS.md declares different ones):

| Document | Default path |
|----------|--------------|
| AGENTS.md | `AGENTS.md` (repo root) |
| PRD | `docs/prd.md` |
| SPEC | `docs/spec/*.md` |
| ADR | `docs/adr/ADR-*.md` |
| IMPL PLAN | `docs/plans/implementation-plan.md` |
| STATUS | `docs/status.md` |
| VERIFICATION LOG | `docs/verification-log.md` |

If `AGENTS.md` exists, read it first — it is the single source of truth and may override everything below.

If `docs/status.md` exists, read it: check the In-Flight Checkpoint (a non-`none` value means the last session crashed — recovery is the first priority) and the latest handoff log entry.

## Step 2: Report the position

Present a compact table: each chain document, whether it exists, and its state (e.g., SPEC count, last STATUS entry date, open task from the latest handoff). Three to five lines of prose maximum around it — this is a briefing, not an essay.

## Step 3: Route

Recommend exactly one next move, based on the first gap in the chain:

| Situation | Route to |
|-----------|----------|
| Substantial existing codebase, but no/partial chain docs | **existing-project-onboarding** — reverse-engineer the as-built chain first (it bootstraps AGENTS.md too) |
| No `AGENTS.md` (little or no code yet) | **agents-md-template** — bootstrap the single source of truth first |
| No PRD and the user has a rough idea | **project-kickoff-prd** — phased dialogue to a prioritized PRD |
| PRD exists, no specs for the target scope | **technical-specification** — harden the PRD into contracts |
| A decision fork is on the table ("X or Y?") | **architecture-decision-record** — capture it before it evaporates |
| Specs approved, no implementation plan | **implementation-plan** — dependency-ordered atomic tasks |
| Plan exists and a session is starting | **status-tracker** — read the briefing, pick the next unblocked task |
| Code work is starting, or a built task needs to ship | **git-workflow** — branch per task, draft PR carrying the done condition |
| A task claims completion / anything sits at 🔍 | **independent-verification** — a fresh-context verdict (bundled `verifier` sub-agent where available) |
| A conversation decision makes a chain doc stale, or the user proposes a new feature / scope change | **documentation-maintenance** — flag the drift, propose the gated update, route the change through the chain |
| In-Flight Checkpoint is not `none` | **status-tracker** — crash recovery from the checkpoint |
| The user wants multiple tasks run unattended | **run-loop** — the chain-aware loop driver (verify-gated, never merges) |

If the user's request names a specific step or goal, weigh it first — route there unless a blocking gap earlier in the chain makes it premature (say so explicitly if it does).

End your briefing with the recommended skill and the first concrete action, then proceed if the user agrees — or immediately, if the route is unambiguous and reversible.
