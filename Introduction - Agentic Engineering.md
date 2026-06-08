# Agentic Engineering — Documentation-First Development

## The Core Problem

**AI Agents are stateless.** Every new session starts from zero — your naming conventions, error handling patterns, architectural constraints are unknown. Without structured documentation, the Agent guesses, causing **context collapse**: code that runs but is architecturally incoherent.

> *The most effective agentic coding workflow starts before the first prompt — with a written spec the agent builds to, not assumptions it makes on its own.* — [Blink](https://blink.new/blog/agentic-coding-best-practices)

## This Skill Suite

A documentation chain that gives Agents full project context from day one, stays current through dynamic updates, and enables seamless handoff between sessions and agents.

**What it solves:**
- Agent produces code that contradicts existing architecture → **SPEC provides the contract**
- Agent makes decisions you already resolved → **ADR preserves decision history**
- New session has no idea what happened last time → **STATUS provides memory**
- Agent builds the wrong thing → **PRD defines what and why**
- Agent does tasks in wrong order or scope → **IMPL PLAN defines the sequence**

---

## Reference Materials

### Development Workflows (pick one)
- [Matt Pocock Skills](https://github.com/mattpocock/skills) — Skills for real engineers
- [Superpowers](https://github.com/obra/superpowers) — Agent skill framework

### Development Principles
- [Karpathy-Inspired Guidelines](https://github.com/multica-ai/andrej-karpathy-skills) — Behavioral guardrails for LLM coding
- [Claude Code Best Practice](https://github.com/shanraisshan/claude-code-best-practice) — Comprehensive reference implementation

---

## The Documentation Chain

Design your high-level architecture upfront through conversation with your Agent. If you need flexibility and extensibility, state it in the first conversation — it shapes every document downstream.

### Execution Order

```
PRD (What & Why) → SPEC (Contract) → ADR (Decisions) → IMPL PLAN (Tasks) → STATUS (Memory)
      ↑                   ↑                ↑                    ↑                  ↑
  Co-creation       Machine-readable   Append-only        Single-session       Live sync
  with Agent         zero-ambiguity    decision log       self-contained       every session
```

### Document Summary

| Doc | Principle | Core Constraint | Skill |
|-----|-----------|-----------------|-------|
| **PRD** | Co-define, think beyond what I say | Agent is a co-creator, not a note-taker | [[project-kickoff-prd]] |
| **SPEC** | Machine-readable, correct on first pass | Any Agent can implement with zero prior context | [[technical-specification]] |
| **ADR** | Append-only, future selves | Never rewrite history — write for the reader 6 months from now | [[architecture-decision-record]] |
| **IMPL PLAN** | Single session, zero additional context | Every task is independently executable | [[implementation-plan]] |
| **STATUS** | Session handoff, living memory | Single source of truth for "where we are" | [[status-tracker]] |

### AGENTS.md — The Entry Point

> See [[agents-md-template]] for the full template.

`AGENTS.md` is the single entry point for any Agent starting a session. It points to all documents in the chain and defines coding conventions, architecture constraints, and boundaries.

**One-line summary:** `AGENTS.md` is the door, `STATUS.md` is memory, `SPEC` is the contract, `ADR` is the law. Agent enters → reads memory → works by contract → doesn't break the law.