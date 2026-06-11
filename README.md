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
- Agent claims "done" when it isn't → **VERIFICATION makes done a verdict, not a claim**
- Crash mid-session loses all progress → **STATUS checkpoints at task granularity**
- Tools each demand their own config → **AGENTS.md is the single entry point**

## Where This Sits — The Harness Layer

[Loop engineering](https://addyosmani.com/blog/loop-engineering/) sits one floor above the harness: automations find the work, worktrees isolate it, sub-agents check it, and the loop feeds itself. This suite is deliberately the floor below — the **memory, contract, and verification substrate** that any loop consumes. It is product-agnostic: the same documentation chain serves Claude Code (`/loop`, scheduled tasks), Codex (Automations, `/goal`), a Ralph-loop bash script, or a human driving sessions by hand.

How the chain maps to the convergent primitives of [long-running agents](https://addyosmani.com/blog/long-running-agents/):

| Long-running agent primitive | Where this suite provides it |
|---|---|
| External completion criteria — "done" defined before work starts | SPEC acceptance criteria (with verification commands) + IMPL PLAN locked done conditions |
| Persistent state outside the context window | STATUS handoff log + In-Flight Checkpoint |
| Independent evaluator — maker-checker separation | [independent-verification](independent-verification/SKILL.md) + `docs/verification-log.md` |
| Checkpoint cadence — every N work units, not only at the end | STATUS checkpoint protocol (task granularity) |
| Project knowledge that survives sessions | AGENTS.md single source of truth |
| Decision history that can't be silently rewritten | ADR (append-only, supersede-only) |

### Loop Safety — Non-Negotiables When a Loop Drives This Chain

- **Circuit breaker** — three consecutive verification FAILs on one task stops the loop and escalates to a human. No infinite fix-verify ping-pong.
- **Test ratchet** — the suite count only goes up. Deleting or skipping tests to go green is an automatic FAIL.
- **Locked done conditions** — the executing agent can never weaken acceptance criteria mid-run; changes require the user (and an ADR if architectural).
- **Human gate** — a machine PASS is necessary, not sufficient: a human reads the diff and can explain it before merge. A loop that outruns your comprehension is accumulating [comprehension debt](https://addyosmani.com/blog/comprehension-debt/), not velocity.

## Layout

This repo is a Claude Code skills collection. Each skill lives in its own directory containing a `SKILL.md`:

```
agentic-engineering/
├── README.md
├── agents-md-template/SKILL.md
├── architecture-decision-record/SKILL.md
├── implementation-plan/SKILL.md
├── independent-verification/SKILL.md
├── project-kickoff-prd/SKILL.md
├── status-tracker/SKILL.md
└── technical-specification/SKILL.md
```

## Installation (Claude Code)

Clone the repo anywhere, then symlink each skill directory into your personal skills directory (`~/.claude/skills/`):

```bash
git clone <this-repo> ~/src/agentic-engineering

for skill in agents-md-template architecture-decision-record implementation-plan \
             independent-verification project-kickoff-prd status-tracker \
             technical-specification; do
  ln -s "$HOME/src/agentic-engineering/$skill" "$HOME/.claude/skills/$skill"
done
```

Each skill self-describes its triggering conditions in the frontmatter `description`, so Claude (or any agent that reads skill metadata) will surface the right one at the right moment.

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
[AGENTS.md (Entry)] → PRD (What & Why) → SPEC (Contract) → ADR (Decisions) → IMPL PLAN (Tasks) → VERIFY (Gate) → STATUS (Memory)
                            ↑                   ↑                ↑                    ↑                 ↑↺                ↑↺
                        Co-creation       Machine-readable   Append-only        Single-session    Fresh-context       Live sync
                         with Agent        zero-ambiguity    decision log       self-contained    verdict, evidence   every session
                                                                                                  (loops every task)  (loops every session)
```

ADRs can be triggered at any point along the chain — whenever a decision fork appears in PRD, SPEC, or IMPL PLAN work. VERIFY loops every task: build → independent verdict → only PASS marks ✅. STATUS loops every session forever.

### Document Summary

| Doc | Principle | Core Constraint | Skill |
|-----|-----------|-----------------|-------|
| **AGENTS.md** | One door, one source of truth | All tool-specific files point here | [agents-md-template](agents-md-template/SKILL.md) |
| **PRD** | Co-define, think beyond what I say | Agent is a co-creator, not a note-taker | [project-kickoff-prd](project-kickoff-prd/SKILL.md) |
| **SPEC** | Machine-readable, correct on first pass | Any Agent can implement with zero prior context | [technical-specification](technical-specification/SKILL.md) |
| **ADR** | Append-only, future selves | Never rewrite history — write for the reader 6 months from now | [architecture-decision-record](architecture-decision-record/SKILL.md) |
| **IMPL PLAN** | Single session, zero additional context | Every task is independently executable | [implementation-plan](implementation-plan/SKILL.md) |
| **VERIFY** | Fresh eyes, evidence or it didn't happen | Producer never grades its own work — done is a verdict | [independent-verification](independent-verification/SKILL.md) |
| **STATUS** | Session handoff, living memory | Single source of truth for "where we are" | [status-tracker](status-tracker/SKILL.md) |

### AGENTS.md — The Entry Point

> See [agents-md-template](agents-md-template/SKILL.md) for the full template.

`AGENTS.md` is the single entry point for any Agent starting a session. It points to all documents in the chain and defines coding conventions, architecture constraints, and boundaries.

**One-line summary:** `AGENTS.md` is the door, `STATUS.md` is memory, `SPEC` is the contract, `ADR` is the law, `VERIFY` is the gate. Agent enters → reads memory → works by contract → doesn't break the law → and never grades its own homework.