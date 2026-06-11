---
name: git-workflow
description: Use when implementation work is about to touch code (branch before the first commit), when a built task needs to ship as a pull request, when defining how branches, commits, and PRs map to plan tasks, or when parallel agents share one repository. Triggers on "create a branch", "commit this", "open a PR", "ship this task", "how do we merge", task start or completion during an implementation plan, worktree setup, or noticing commits landing directly on main.
---

# Git Workflow — Shipping Through the Gate

> **Core constraint:** `main` is read-only for agents. Work lands on task branches, ships as draft PRs carrying verification evidence, and merges only after the verifier's PASS and a human who has read the diff. The agent never merges its own PR.

## Quick Reference

| | |
|---|---|
| **Use when** | Code work starts (branch first); a task is built and needs to ship; parallel agents share a repo |
| **Skip when** | Doc-only edits the user has exempted, or repos where the user explicitly owns all git operations |
| **Output** | One branch + one draft PR per task, the PR description carrying the done condition and verdict reference |
| **Sequence** | IMPL PLAN task starts → **BRANCH** → build → self-check → **DRAFT PR** → VERIFY → PASS → ready → human gate → **MERGE** → STATUS ✅ |
| **Iron rule** | One task, one branch, one PR. Never commit to main. Never merge your own PR. |
| **Sibling skills** | [[project-kickoff-prd]] · [[technical-specification]] · [[architecture-decision-record]] · [[implementation-plan]] · [[status-tracker]] · [[independent-verification]] · [[agents-md-template]] |

## Why This Exists

The rest of the chain defines what to build (IMPL PLAN), how to prove it (VERIFY), and how to remember it (STATUS) — but without a git contract, how code actually lands is left to agent defaults, and agent defaults are destructive: commits straight to main, three tasks bundled into one diff, "done" declared with nothing pushed.

Two concepts the chain repeats — *merging through the verification gate* and the *human gate* — only become operational when there is a branch to verify and a PR for the human to read. This skill is that contract.

## Branch Protocol

1. **main is protected.** Agents never commit to it, never push to it, never rewrite it.
2. **One branch per task**, created from up-to-date main at task start (the branch is the workspace — the In-Flight Checkpoint references it):
   - Naming: `task/<task-id>-<slug>` — e.g. `task/m1-t2-user-model`
   - For long milestones, an optional integration branch `milestone/<id>` — task branches then fork from and PR into it
3. **Never reuse a branch across tasks.** Never let a task branch outlive its merge.

## Commit Protocol

1. Small commits inside the task — each a coherent step, not an end-of-session dump.
2. Message format: `<task-id>: <imperative summary>` (e.g., `M1-T2: add user model with validation`). Cite the spec section in the body when it clarifies intent.
3. No drive-by changes. An unrelated fix discovered mid-task becomes its own task in the plan, not a stowaway commit.
4. **No history rewriting once verification has started.** The verdict cites a commit range; force-pushing invalidates the evidence. If a rebase is unavoidable, the task drops back to 🔍 and re-verifies.

## PR Protocol

1. Producer self-check passes (lint, full suite, test ratchet) → push the branch → **open a draft PR immediately.** The diff becomes inspectable the moment it exists.
2. The PR description carries the contract:

   ```markdown
   **Task:** M1-T2 — [title] ([plan link])
   **Spec:** spec/auth.md §2.1
   **Done condition:** [the executable criteria, copy-pasteable]
   **Verification:** [link to the verdict entry in docs/verification-log.md — added when the verdict lands]
   **Caveats:** ⚠️ [anything that needs special environment / keys / follow-up]
   ```

3. **One task = one PR.** A PR that needs a second task's changes is mis-scoped — fix the plan, not the PR.
4. **Draft until PASS.** Flip to "ready for review" only when the verifier's PASS verdict is recorded, adding the verdict reference to the description in the same motion.
5. After a FAIL verdict: fix commits go to the same branch, each round noted in the PR thread (`round 2/3`). Silent fix-ups are an anti-pattern — the PR history must match the verdict history.

## Merge Gate

Merging requires **all three**:

1. Verifier PASS recorded in `docs/verification-log.md`
2. A human has read the diff and can explain it (the human gate)
3. CI green, if CI exists — noting that CI green alone is neither ① nor ②

**The agent never merges its own PR.** In unattended loops this is absolute: the loop's output is a queue of verified, evidence-carrying PRs — merged code is always a human's move.

After merge: STATUS flips the module to ✅ with PR + verdict references, and the task branch is deleted — only after merge; until then the branch is evidence.

## Worktrees — Parallel Agents

- One worktree per agent: `git worktree add ../<repo>-<task-id> task/<task-id>-<slug>` — isolated working copies of the same repo, no file collisions.
- Worktrees solve mechanical conflicts only. The real ceiling is human review bandwidth: every parallel agent emits PRs a human must read, and verdicts a verifier must produce.
- Tasks touching the same files must not run in parallel regardless of worktrees — that is a dependency the plan missed (→ [[implementation-plan]]).
- Remove the worktree after merge: `git worktree remove ../<repo>-<task-id>`.

## Rules

1. **main is read-only for agents.** No exceptions, including "trivial" fixes.
2. **One task, one branch, one PR.** Traceability runs diff → PR → task → spec.
3. **Branch names derive from task IDs.** A reviewer can find the contract from the branch name alone.
4. **PRs are born draft.** Ready-for-review is earned by a PASS verdict, not by the producer's confidence.
5. **No history rewriting after verification starts.** Evidence integrity outranks a tidy log.
6. **Never merge your own PR.** Both gates (verifier + human) live on the merge button.
7. **Delete branches only after merge.** Branches referenced by verdicts are evidence.
8. **AGENTS.md wins.** Repo-specific overrides (protected branch names, PR conventions, merge strategy) live in `AGENTS.md` — this skill defines the defaults.

## Anti-Patterns (Do Not)

| Anti-Pattern | Why It Fails |
|--------------|--------------|
| Committing to main "just this once" | The exception becomes the workflow; one stray commit on main poisons every open branch |
| Multi-task PRs | The reviewer can't map the diff to a done condition; verdicts lose their object |
| Silent fix-ups after a FAIL | Unannounced commits or force-pushes decouple the PR from the verdict history |
| Merging because CI is green | CI is necessary, never sufficient — it is neither the verifier nor the human gate |
| Long-lived task branches | A task is ≤ 90 minutes by plan rule; a week-old branch means the task was mis-scoped |
| Deleting branches before merge | Destroys the evidence the verdict references |
| Agent merging its own PR | The one unguarded door that bypasses both gates at once |

## Relationship to Other Documents

- **IMPL PLAN** supplies the task IDs that name branches and PRs ([[implementation-plan]])
- **VERIFY** verdicts cite the branch/commit range; PASS is what flips a PR from draft to ready ([[independent-verification]])
- **STATUS** records PR state per module and the verdict reference behind every ✅ ([[status-tracker]])
- **AGENTS.md** carries repo-specific git overrides and the boundaries that enforce this contract ([[agents-md-template]])
