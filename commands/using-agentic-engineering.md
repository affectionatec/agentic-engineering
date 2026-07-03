---
description: Entry point for the Agentic Engineering suite. Assesses which documentation-chain documents exist in this project, reports where the project stands, and routes to the right skill for the next step.
argument-hint: [optional — what you want to do next]
---

Invoke the `agentic-engineering:using-agentic-engineering` skill, weighing these arguments first if present: $ARGUMENTS

The skill carries the full router protocol — assess the chain documents, report the position, and route to exactly one next skill. Follow it exactly. (This command is a thin alias so `/using-agentic-engineering` works in Claude Code; the skill is the single source of truth and also serves harnesses that load skills but not plugin commands.)
