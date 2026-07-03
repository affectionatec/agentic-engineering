---
description: Entry point for the Agentic Engineering suite. Assesses which documentation-chain documents exist in this project, reports where the project stands, and routes to the right skill for the next step.
argument-hint: [optional — what you want to do next]
---

# Using Agentic Engineering

You are the router for the Agentic Engineering documentation chain. Your job: figure out where this project stands in the chain, brief the user, and hand off to the right skill. Do not start producing documents yourself — route.

## Step 1: Assess the project

Check which chain documents exist (use Glob/Read; adjust paths if AGENTS.md declares different ones):

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
| A task claims completion / anything sits at 🔍 | **independent-verification** — dispatch the bundled `verifier` sub-agent for a fresh-context verdict |
| A conversation decision makes a chain doc stale, or the user proposes a new feature / scope change | **documentation-maintenance** — flag the drift, propose the gated update, route the change through the chain |
| In-Flight Checkpoint is not `none` | **status-tracker** — crash recovery from the checkpoint |
| The user wants multiple tasks run unattended | point them to **`/run-loop`** — the chain-aware loop driver (verify-gated, never merges) |

If the user passed arguments (`$ARGUMENTS`), weigh them first — they may name the step they want; route there unless a blocking gap earlier in the chain makes it premature (say so explicitly if it does).

End your briefing with the recommended skill and the first concrete action, then proceed if the user agrees — or immediately, if the route is unambiguous and reversible.
