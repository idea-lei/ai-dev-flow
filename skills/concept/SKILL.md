---
name: concept
description: "Build or challenge an implementation concept for a ticket or PR. Use when triage says a concept is needed, when asked for a concept, design or approach comparison, or when a PR's mechanism looks heavier than the problem."
---

# Concept

A concept is valid only when its loop is closed: every path that can break the goal is checked. Read the code first; don't change anything until the developer agrees.

## Method

1. **Invariant.** What must be true after every operation, stated observably. Can't state it -> ask.
2. **Ownership.** Source of truth or replica? Owned data can be re-read; replicas only have messages.
3. **Earn every part, in this order** (Musk's algorithm: optimizing a part that should not exist is the common error):
   - **Source.** Name who wants each requirement: ticket text, product owner, developer, agent, borrowed pattern. The developer's own ideas included; borrowed patterns: same reason here?
   - **Delete.** Name the failure that occurs without the part, traced in code ("rare" is not evidence). No failure -> propose dropping it; beyond the ticket but useful -> propose it as optional (e.g. a manual reprocess endpoint).
   - **Simplify** only what survives; a mechanism only for a named failure rated by the risk matrix in `../branch-review/code-review.instructions.md`.
   Bring the result to the developer as a closed proposal (evidence, recommendation), never as an open question; ask only what the code can't answer (intent, priority, product scope).
4. **Close the loop.** Check every path: all writers, subscribers/retries/jobs, partial failure, existing rows, consumers. Each one: holds / accepted (matrix cell) / open (owner). Nothing left "to settle".
5. **Options** only if there is a real choice. Research what exists first: our code, libraries, outside practice. Compare the viable ones on fit, effort (total cost: build, integrate, operate), performance, maintenance and risk (failure modes, migration, consumers); probe the claims that decide it rather than trusting them. Include the cons of the recommended one. A choice with lasting trade-offs -> `adr`. Why: cheap code makes the first plausible approach tempting; the expensive mistake is the approach, not the code. Example: a `Library Selection.md` in the task folder.
6. **Tests.** Which test proves each invariant.

To challenge someone else's concept or PR: run 1-4 on it and report only the gaps.

## Output (one screen)

```text
Invariant:
Ownership:
Approach:
Loop: | Path | holds / accepted / open | why |
Options: (if any)
Tests:
Open: question -> owner
```

Then stop. The developer decides and keeps the decision, the rejected options and the accepted trade-off in their design notes (the task folder, see `ai-dev-steps` "Where records go"; on request write the output there as `Concept.md`); the PR's why-bullets name the chosen approach. The tracker holds the WHAT, not the design.
