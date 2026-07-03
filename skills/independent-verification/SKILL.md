---
name: independent-verification
description: Use when a task or milestone claims completion and must be verified before being marked done, when acting as a verifier sub-agent checking a producer's work, or when a "done" claim needs evidence. Triggers on "verify this task", "is this actually done", "acceptance check", "run verification", any completion claim, any 🔍 → ✅ status transition, or dispatching a maker-checker review loop.
---

# Independent Verification — The Maker-Checker Gate

> **Core constraint:** The agent that wrote the code never grades it. "Done" is a verdict returned by a verifier with fresh context — not a claim made by the producer.

## Quick Reference

| | |
|---|---|
| **Use when** | A task or milestone claims completion; before any task is marked ✅ in STATUS |
| **Skip when** | Throwaway prototyping the user has explicitly exempted, or pure doc typo fixes |
| **Output** | Verdict entry appended to `docs/verification-log.md` (PASS / FAIL per criterion, with evidence) |
| **Sequence** | IMPL PLAN task built → **VERIFY** → STATUS records ✅ (loops every task) |
| **Iron rule** | Fresh context. The verifier never sees the producer's conversation, and never edits code. |
| **Sibling skills** | [[project-kickoff-prd]] · [[technical-specification]] · [[architecture-decision-record]] · [[implementation-plan]] · [[status-tracker]] · [[git-workflow]] |

## Why This Exists

The deadliest failure mode of an autonomous coding session is not a crash — it is **claiming completion without completing**. Models grade their own homework too generously: the same model that wrote the code will rate it "looks good" every time, and an agent without an external gate declares "done" at 30% completion with full confidence.

The fix is structural, not behavioral. You cannot prompt a producer into objectivity — you separate the roles:

1. The **done condition** is written down before work starts (IMPL PLAN task + SPEC acceptance criteria). The producer cannot redefine "done" mid-run.
2. The **producer** builds and self-checks. Self-check is necessary, never sufficient.
3. The **verifier** — fresh context, different conversation, optionally a different model — executes the done condition as written and returns a verdict with evidence.
4. Only a **PASS verdict** moves the task to ✅ in STATUS.

## Roles

| Role | Does | Must NOT |
|------|------|----------|
| **Producer** | Implements the task, runs self-checks (lint, tests), assembles the evidence package | Mark its own task done · edit acceptance criteria · write the verdict |
| **Verifier** | Reads the done condition first, re-runs every verification command, inspects the diff, writes the verdict | Fix code · inherit the producer's conversation · negotiate or reinterpret criteria |

**Fresh context is the whole point.** Run the verifier as a sub-agent with no shared conversation, a fresh session, or a separate agent entirely. A verifier that saw the producer's reasoning inherits the producer's blind spots — that is verification theater, not verification.

> **If this suite is installed as the Claude Code plugin**, a ready-made verifier ships with it: dispatch the bundled `verifier` sub-agent (explicitly via `@agent-agentic-engineering:verifier`, or let delegation pick it up). Sub-agents start with fresh context by construction — the isolation this skill requires is structural, not a discipline to remember.

## Verifier Protocol

### Step 1: Read the contract — before reading the code

In this order:

1. The task's done condition (IMPL PLAN task entry — acceptance criteria + declared file list)
2. The SPEC acceptance criteria the task references (each carries its verification command)
3. `AGENTS.md` — canonical test/lint/build commands and boundaries

Reading the criteria before the diff prevents the verifier from rationalizing what it sees.

### Step 2: Inspect the diff

- Identify every file created or modified
- **Scope check:** does the diff stay within the task's declared file list? Flag any drift.
- **Test ratchet check:** were any tests deleted, skipped, or weakened? Test count must not decrease. Deleting tests to make a run pass is an automatic FAIL.

### Step 3: Execute the done condition

- Run every verification command exactly as written in the SPEC / IMPL PLAN. No interpretation, no substitution.
- Record exit codes and the key output lines — evidence, not impressions.
- Run the full test suite, not just the new tests. Report exact counts and compare against the baseline from the previous verdict.

### Step 4: Render the verdict

- Verdict is **binary per criterion**: PASS or FAIL. "Mostly works" is FAIL.
- Overall verdict is PASS only if every criterion passes, the ratchet holds, and no unexplained scope drift exists.
- For failures: state what was expected vs. what was observed. Locate the gap; do not write the fix.

## Verdict Template

> Append to `docs/verification-log.md`, newest first. Entries are append-only — never edit a past verdict.

````markdown
## YYYY-MM-DD — [Task ID]: [Task Title] — VERDICT: PASS | FAIL

> Verifier context: fresh session/sub-agent · Diff: [commit range or branch]

**Test ratchet:** baseline [N] → now [M] (✅ holds / ❌ decreased)
**Scope:** clean / ⚠️ drift — [files touched outside the task's declared list]

**Commands executed:**

| Command | Exit | Key output |
|---------|------|-----------|
| `pytest tests/ -v` | 0 | 147 passed (baseline 142, +5 new) |
| `ruff check` | 0 | clean |

**Criteria:**

| # | Criterion | Verdict | Evidence |
|---|-----------|---------|----------|
| 1 | Given [precondition] When [action] Then [outcome] | ✅ PASS | `pytest tests/test_auth.py::test_expiry` 1/1 |
| 2 | Given [edge case] When [action] Then [handling] | ❌ FAIL | expected 409 CONFLICT, observed 500 |

**Failure detail (if any):** [expected vs. observed, file/line of the gap — no fix proposals]
**Round:** [1/3] [if FAIL: producer fixes and resubmits]
````

## Loop Rules (Producer ↔ Verifier)

1. **FAIL → producer fixes → re-verify.** Each round gets a new verdict entry. The fix round only addresses the failed criteria; the verifier still re-runs the full suite (regressions hide in fixes).
2. **Circuit breaker: three rounds maximum.** After the third consecutive FAIL on the same task, stop. Escalate to the user with the verdict history. A loop that cannot converge is burning tokens on a problem that needs a human decision — usually the task or spec is wrong, not the code.
3. **Criteria are locked while the loop runs.** Weakening a criterion to make it pass requires explicit user approval — and an ADR if the change is architectural. The producer asking the verifier to "skip that one" is the exact failure this skill exists to prevent.
4. **The test ratchet is non-negotiable.** Suite count only goes up. A deleted or skipped test is an automatic FAIL regardless of other criteria.
5. **Only PASS moves STATUS to ✅.** The STATUS module table references the verdict (date + task ID). A ✅ without a verdict entry is a claim, not a completion.

## The Human Gate (After the Machine Verdict)

A machine PASS is necessary — not sufficient — for merge:

- **A human reads the diff before merge** and must be able to explain the change in their own words. If you can't explain it, you don't own it — you've accumulated comprehension debt that compounds with every merge.
- The verdict entry doubles as the human's review digest: what changed, what was verified, what the risks were.
- The faster the loop produces code, the more this gate matters. An unattended loop is also a loop making unattended mistakes — verification gives the "done" claim weight, but reading what your agent wrote is what keeps you the engineer.

## Cost Discipline

Independent verification doubles model invocations per task. Spend it where it pays:

- **Always verify:** plan tasks, anything merging to main, anything touching data, auth, money, or public APIs.
- **User may exempt:** throwaway spikes, prototypes explicitly marked disposable.
- A cheaper/faster model is acceptable as verifier for mechanical checks (commands, counts, diffs); use a strong model when criteria require judgment.

## Anti-Patterns (Do Not)

| Anti-Pattern | Why It Fails |
|--------------|--------------|
| Verifier shares the producer's session | Inherits the same blind spots — verification theater |
| Verifier fixes the code it's checking | It just became a producer; the verdict is now self-graded |
| "Looks good" without commands run | A verdict without evidence is a claim with extra steps |
| Re-litigating the spec during verification | The gate checks compliance, not design. Design concerns → flag for ADR, verdict stays on the written criteria |
| Letting the producer pre-summarize "what to check" | The done condition on disk is the input — not the producer's framing of it |
| Infinite fix-verify ping-pong | No convergence = wrong task or wrong spec. Circuit breaker at 3 rounds, escalate |
| Marking ✅ on a producer claim | The exact failure mode this skill exists to prevent |

## Relationship to Other Documents

- **IMPL PLAN** defines each task's done condition — the verifier executes it as written ([[implementation-plan]])
- **SPEC** holds acceptance criteria with verification commands per criterion ([[technical-specification]])
- **STATUS** records ✅ only with a verdict reference; the 🔍 state marks "built, awaiting verification" ([[status-tracker]])
- **GIT WORKFLOW** puts the work on a branch and a draft PR before verification; a PASS verdict is what flips the PR from draft to ready ([[git-workflow]])
- **AGENTS.md** defines the canonical commands the verifier runs ([[agents-md-template]])
- **ADR** captures any criteria change that survives escalation ([[architecture-decision-record]])
