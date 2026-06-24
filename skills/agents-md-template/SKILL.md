---
name: agents-md-template
description: Use when bootstrapping a new repo for AI-assisted development, when no AGENTS.md / CLAUDE.md / copilot-instructions.md / .cursorrules exists yet, or when consolidating scattered tool-specific config into one source of truth. Triggers on "create AGENTS.md", "set up agent instructions", "new project setup", "configure Claude/Cursor/Copilot/Codex", "onboarding a repo", or noticing duplicated agent config across tool-specific files.
---

# AGENTS.md Template

> **Design principle:** `AGENTS.md` is the single source of truth. Tool-specific files (CLAUDE.md, copilot-instructions.md, .cursor/rules/) all point here. — [substratia.io](https://substratia.io/blog/agents-md-vs-claude-md/)

## Quick Reference

| | |
|---|---|
| **Use when** | Bootstrapping a repo for AI-assisted dev; no AGENTS.md exists yet |
| **Skip when** | AGENTS.md already exists — edit in place; don't recreate |
| **Output** | `AGENTS.md` at repo root + one-line pointer files for each tool |
| **Sequence** | Step 0 — before any documentation chain work begins |
| **Iron rule** | One source of truth. CLAUDE.md / .cursor / copilot all point here. |
| **Sibling skills** | [[project-kickoff-prd]] · [[technical-specification]] · [[architecture-decision-record]] · [[implementation-plan]] · [[status-tracker]] · [[independent-verification]] · [[git-workflow]] |

---

## Template

> Copy everything below into your project root as `AGENTS.md`, then fill in the brackets.

````markdown
> **Stop. Read this entire file before doing anything.**
> This is the single source of truth for how we work on this project.
> Tool-specific files (CLAUDE.md, copilot-instructions.md, .cursor/rules/)
> all point here. Do not look for instructions elsewhere.

## 1. Project Identity
- **Name:** [Project Name]
- **One-liner:** [What this project does in one sentence]
- **Tech stack:** [e.g., Python 3.12 / FastAPI / Terraform / GitHub Actions]
- **Repo structure:**

## 2. Documentation Chain — Read Before You Code

This project uses a six-document chain. **Read them in order.**

| Doc | Path | Purpose | When to Read |
|-----|------|---------|-------------|
| **PRD** | `docs/prd.md` | What we're building and why | Before any feature work |
| **SPEC** | `docs/spec/*.md` | Precise technical contracts (API, data model, edge cases) | Before writing any code |
| **ADR** | `docs/adr/ADR-*.md` | Why we chose A over B — append-only, never rewrite | Before making architectural decisions |
| **IMPL PLAN** | `docs/plans/implementation-plan.md` | Milestones and step-by-step tasks | Before starting a task |
| **STATUS** | `docs/status.md` | Where we are right now — live progress | **First thing every session** |
| **VERIFICATION LOG** | `docs/verification-log.md` | Verdicts with evidence for every completed task — append-only | Before marking anything done; when auditing what "done" meant |

### Session Protocol

1. **Start of session:** Read `docs/status.md`. If the In-Flight Checkpoint is not `none`, the previous session crashed — recover from it. Otherwise the latest handoff log entry is your briefing.
2. **Before coding:** Read the SPEC for the module you're working on. Follow the contract exactly.
3. **Decision point:** Check `docs/adr/` before making any architectural choice. If no ADR covers it, flag it to the user.
4. **Documentation drift:** If anything decided in this session makes a fundamental document (PRD, SPEC, ADR, IMPL PLAN, this file) stale or contradicted — or the user proposes a new feature — flag it the moment it happens and propose the update. **Never edit a fundamental document without the user's explicit approval.** Route the change through the chain before writing code.
5. **After each completed task:** Checkpoint `docs/status.md` (In-Flight Checkpoint + module table). Push the task branch and open a draft PR, then request independent verification — the task stays at 🔍 until a verifier with fresh context returns PASS. Never mark your own work ✅; never merge your own PR.
6. **End of session:** Update `docs/status.md` — module table, header block, append a handoff log entry, reset the In-Flight Checkpoint to `none`.

---

## 3. Coding Conventions

- **Language:** [e.g., Python — strict typing, all functions annotated]
- **Formatter:** [e.g., ruff format]
- **Linter:** [e.g., ruff check --fix]
- **Test framework:** [e.g., pytest]
- **Test command:** [e.g., `pytest tests/ -v`]
- **Build command:** [e.g., `docker compose build`]
- **Pre-commit checks:** [e.g., `ruff check && ruff format --check && pytest`]

---

## 4. Architecture Constraints (Quick Reference)

> Full rationale in `docs/adr/`. **Do not relitigate** — raise changes with the user.

- [e.g., Cloud-neutral orchestration; cloud-specific only at the provider layer (ADR-001)]
- [e.g., All network calls isolated behind a single function for offline testability (ADR-003)]
- [e.g., Point-in-time correctness: never read data filed after the decision date (ADR-005)]

---

## 5. Boundaries — What You Must NOT Do

- **Do not create files outside the defined structure** without asking.
- **Do not install new dependencies** without asking.
- **Do not modify ADRs.** They are append-only. Supersede with a new ADR if needed.
- **Do not edit a fundamental document** (PRD, SPEC, ADR, IMPL PLAN, AGENTS.md) without the user's explicit approval — propose the change, then wait for the nod.
- **Do not skip the status update.** Every session ends with a handoff log entry.
- **Do not guess when the spec is ambiguous.** Ask the user.
- **Do not grade your own work.** Marking a task ✅ requires an independent verifier's PASS, not your claim.
- **Do not delete, skip, or weaken tests to make a run pass.** The suite count only goes up (test ratchet). A failing test is information, not an obstacle.
- **Do not redefine "done" mid-task.** Acceptance criteria are locked when the task starts; changes require the user's explicit approval.
- **Do not commit directly to main.** Every task gets its own branch and ships as a draft PR (one task = one PR).
- **Do not merge your own PR.** Merge requires the verifier's PASS and a human who has read the diff.

---

## 6. Verification — Self-Check, Then Independent Gate

### 6.1 Producer self-check (necessary, never sufficient)

Before claiming a task is built:

1. **Lint clean:** `[lint command]` exits 0
2. **Tests pass:** `[test command]` — report exact count (e.g., "142/142 pytest")
3. **No regressions:** all pre-existing tests still pass
4. **Test ratchet holds:** suite count ≥ previous baseline — no tests deleted or skipped
5. **Spec compliance:** every acceptance criterion in the relevant spec is met
6. **Status updated:** `docs/status.md` reflects what you just did

### 6.2 Independent verification (the gate)

- A verifier with **fresh context** — sub-agent or fresh session, never this conversation — re-runs the task's done condition exactly as written in the SPEC / IMPL PLAN and appends a verdict with evidence to `docs/verification-log.md`.
- Only a PASS verdict moves the task from 🔍 to ✅ in STATUS.
- Three consecutive FAILs on the same task → stop and escalate to the user. Do not loop indefinitely; do not weaken criteria to converge.

### 6.3 Human gate (before merge)

- A human reads the diff and can explain the change in their own words before it merges. Code you can't explain is comprehension debt — and it compounds.
- The verification log entry is the review digest: what changed, what was proven, what the risks are.

---

## Full Chain Summary

```
Agent Start 
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│  CLAUDE.md / copilot-instructions.md / .cursorrules     │
│  → "Read AGENTS.md. It's the source of truth."          │
└────────────────────────┬────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────┐
│  AGENTS.md  ← 🎯 The one file that grabs attention      │
│                                                         │
│  §1 Project Identity (what is this?)                    │
│  §2 Documentation Chain (where to find context)         │
│     → PRD → SPEC → ADR → IMPL PLAN → STATUS             │
│  §3 Coding Conventions (how to write code)              │
│  §4 Architecture Constraints (what's already decided)   │
│  §5 Boundaries (what NOT to do)                         │
│  §6 Verification (self-check → independent gate)        │
└────────────────────────┬────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────┐
│  docs/status.md  ← Agent first move                     │
│  "Read the latest handoff log. That's your briefing."   │
└────────────────────────┬────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────┐
│  docs/spec/*.md  ← must read before coding              │
│  "Follow the contract. Ambiguity = ask, don't guess."   │
└────────────────────────┬────────────────────────────────┘
                         ▼
                  Build the task.
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Independent verification  ← fresh context, not you     │
│  Verifier re-runs the done condition, appends verdict   │
│  to docs/verification-log.md. Only PASS marks ✅.       │
└────────────────────────┬────────────────────────────────┘
                         ▼
        Checkpoint docs/status.md. Next task or hand off.
```
````

---

## How Tool-Specific Files Should Reference AGENTS.md

Each AI coding tool has its own config file. They should all contain only one instruction:

**CLAUDE.md:**
```
Read and follow AGENTS.md in the project root. It is the single source of truth.
```

**`.github/copilot-instructions.md`:**
```
Read and follow AGENTS.md in the project root. It is the single source of truth.
```

**`.cursor/rules/main.mdc`:**
```
Read and follow AGENTS.md in the project root. It is the single source of truth.
```

This ensures any Agent, regardless of tool, gets the same instructions.