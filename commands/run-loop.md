---
description: Chain-aware loop driver. Works through implementation-plan tasks unattended — branch, build, ship a draft PR, dispatch the verifier, checkpoint, repeat — until the target is reached, the circuit breaker trips, or the task budget is spent. Never merges.
argument-hint: [milestone e.g. M2 | task ID | until-blocked] [max-tasks=N]
---

Invoke the `agentic-engineering:run-loop` skill with this target: $ARGUMENTS

The skill carries the full loop protocol — target parsing, preconditions, the per-task loop body, stop conditions, wind-down, and hard rules. Follow it exactly. (This command is a thin alias so `/run-loop` works in Claude Code; the skill is the single source of truth and also serves harnesses that load skills but not plugin commands.)
