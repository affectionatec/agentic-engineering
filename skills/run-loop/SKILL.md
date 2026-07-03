---
name: run-loop
description: Chain-aware unattended loop driver for the Agentic Engineering suite. Use ONLY when the user explicitly asks to run multiple implementation-plan tasks unattended — "run the loop", "run-loop M2", "work through milestone M2 without stopping", "run the next N tasks", "keep going until blocked". Never self-trigger for a single task or an ordinary coding request. Branch → build → draft PR → dispatch the verifier → checkpoint → repeat, until the target is reached, the circuit breaker trips (3 FAILs on one task), or the task budget is spent. Never merges.
---

# Run Loop

Drive the documentation chain end-to-end without per-task prompting. You are the producer; verification is dispatched to a fresh-context verifier, never self-graded; merging is never yours.

## Target

Parse the user's instruction:

- A milestone (e.g. `M2`) → run its tasks in dependency order until the milestone is done
- A task ID (e.g. `M2-T3`) → run exactly that task
- `until-blocked` → keep taking the next unblocked task until nothing is runnable
- `max-tasks=N` → cap on tasks this run (default **5**; the user must raise it explicitly for longer runs)

No target given → ask for one before looping. Do not guess.

## Preconditions — abort with a clear message if unmet

1. `AGENTS.md`, `docs/plans/implementation-plan.md`, and `docs/status.md` exist. If not, stop and route to the `using-agentic-engineering` router — a loop without the chain is just unsupervised guessing.
2. Working tree is clean and `main` is up to date.
3. The In-Flight Checkpoint in `docs/status.md` is `none`. If it isn't, the previous session crashed — run the status-tracker recovery protocol first, then loop.

## Loop body — per task

1. **Pick** the next unblocked task for the target (STATUS module table + plan dependency order).
2. **Branch** per the git-workflow skill: `task/<task-id>-<slug>` from up-to-date main.
3. **Build** to the spec. Producer self-check: lint clean, full suite passing, test ratchet holds.
4. **Checkpoint** `docs/status.md`: In-Flight Checkpoint updated, module table → 🔍.
5. **Ship**: push the branch, open a draft PR whose description carries the task reference and done condition (git-workflow PR protocol).
6. **Verify**: dispatch a fresh-context verifier with the task ID, branch name, and done-condition location — the bundled `verifier` sub-agent where the harness supports it, otherwise any fresh-context mechanism per the independent-verification skill (a fresh session or separate agent). Never verify your own work in this conversation.
7. **On PASS**: STATUS module table → ✅ with verdict + PR references; flip the PR from draft to ready; proceed to the next task.
8. **On FAIL**: fix on the same branch (each round noted in the PR), re-dispatch the verifier. **Three consecutive FAILs on one task stops the entire loop** — escalate to the user with the verdict history.

## Stop conditions — whichever comes first

- Target reached
- Circuit breaker: 3 consecutive FAILs on one task
- `max-tasks` budget spent
- No unblocked tasks remain
- A precondition broke mid-run (e.g. the tree turned dirty outside your work)

## Wind-down — always, even on abort

1. Finalize `docs/status.md`: module table, header block, handoff log entry; In-Flight Checkpoint reset to `none`.
2. Report: tasks completed (with PR + verdict references), tasks failed (with the last verdict), tasks remaining, and exactly where and why the loop stopped.

## Hard rules

- **Never merge.** The loop's output is a queue of verified draft→ready PRs. Merging belongs to the human gate, after a human has read each diff.
- Never weaken a done condition to make the loop converge — that is escalation, not negotiation.
- Checkpoint before every risky step. A crash must cost at most one task.
- AGENTS.md boundaries apply to every iteration — the loop is not a license to skip the chain.
