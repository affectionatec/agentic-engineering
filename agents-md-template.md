---
name: agents-md-template
description: Use when setting up a new project's agent configuration. Triggers on "create AGENTS.md", "set up agent instructions", "new project setup", or when no AGENTS.md exists yet. Produces the single entry-point file that all AI coding tools read first.
---

# AGENTS.md Template

> **Design principle:** `AGENTS.md` is the single source of truth. Tool-specific files (CLAUDE.md, copilot-instructions.md, .cursor/rules/) all point here. — [substratia.io](https://substratia.io/blog/agents-md-vs-claude-md/)

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

This project uses a four-document chain. **Read them in order.**

| Doc | Path | Purpose | When to Read |
|-----|------|---------|-------------|
| **PRD** | `docs/prd.md` | What we're building and why | Before any feature work |
| **SPEC** | `docs/spec/*.md` | Precise technical contracts (API, data model, edge cases) | Before writing any code |
| **ADR** | `docs/adr/ADR-*.md` | Why we chose A over B — append-only, never rewrite | Before making architectural decisions |
| **IMPL PLAN** | `docs/plans/implementation-plan.md` | Milestones and step-by-step tasks | Before starting a task |
| **STATUS** | `docs/status.md` | Where we are right now — live progress | **First thing every session** |

### Session Protocol

1. **Start of session:** Read `docs/status.md`. Find the latest handoff log entry. That's your briefing.
2. **Before coding:** Read the SPEC for the module you're working on. Follow the contract exactly.
3. **Decision point:** Check `docs/adr/` before making any architectural choice. If no ADR covers it, flag it to the user.
4. **End of session:** Update `docs/status.md` — module table, header block, and append a handoff log entry.

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
- **Do not skip the status update.** Every session ends with a handoff log entry.
- **Do not guess when the spec is ambiguous.** Ask the user.

---

## 6. How to Verify Your Work

Before marking any task as done:

1. **Lint clean:** `[lint command]` exits 0
2. **Tests pass:** `[test command]` — report exact count (e.g., "142/142 pytest")
3. **No regressions:** all pre-existing tests still pass
4. **Spec compliance:** every acceptance criterion in the relevant spec is met
5. **Status updated:** `docs/status.md` reflects what you just did

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
│  §6 Verification (how to prove it's done)               │
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
                    Start coding.
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