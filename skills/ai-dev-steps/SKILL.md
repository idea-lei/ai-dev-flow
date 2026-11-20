---
name: ai-dev-steps
description: "Working steps for ticket work - my own tickets and reviews of a colleague's branch or PR - with a per-task folder. Load FIRST in any chat that works on a ticket (branch name or text contains a ticket key, or a branch/PR to review), and when asked 'where are we', 'what's next', or before review/PR. Not for questions unrelated to a ticket."
---

# AI dev steps

The agent drives these steps; the developer may override any of them. This skill orchestrates: load each step's tool when entering the step. The rating standard is `../branch-review/code-review.instructions.md`.

## Principles

Use them where a step doesn't cover the case.

1. A person decides; the agent proposes with evidence. Why: AI is often right and sometimes confidently wrong.
2. Done is shown, not claimed: the output, not "tests pass".
3. Independence finds blind spots: blind review in a fresh context; each finding verified by a second agent or probe.
4. Loops converge, or go back to the step that owns the broken assumption. Why: patching downstream never ends.
5. Every part earns its place. Why: cheap code makes it easy to build what nobody needs.
6. Depth follows risk; findings follow impact. Why: attention is the scarce resource.

## Locations

Notes root `<notes-root>` (local git; the developer commits), one folder per project:

| Path | Holds |
|---|---|
| `<project>/Tasks/<YY.MM> [<KEY>-123] <topic>` | active task folders |
| `<project>/Tasks/Archive/` | task folders after merge |
| `<project>/ADRs/` | design decisions (`adr`) |

`<project>` is the folder for the current repo; unclear -> ask once. Other folders under the root are not part of ticket work.

## Start

1. Ticket key from the request first, then the branch name (the checked-out branch may belong to another ticket). No ticket is fine: then the topic is the key. Unclear -> ask once.
2. Role: author (my ticket) or reviewer (a colleague's branch or PR). Unclear -> ask.
3. Entry: the step the request or the branch state points to ("review PR #n" -> 5, "check the comments" -> 9); earlier steps are not required. Why: work often starts mid-way.
4. Author without a branch for the ticket: create `feature/<KEY>-123-<short-slug>` from the integration branch (e.g. `develop`) and check it out. Uncommitted changes are carried only from the integration branch; uncommitted changes on another branch -> ask first. Standing permission; commits stay with the developer. Why: work once landed directly on the integration branch; git carries edits silently, so another ticket's edits would move along.
5. Find the task folder in `Tasks/` and `Tasks/Archive/` (Locations) by ticket key; without a ticket, by topic words or the branch slug; several match -> ask once. Missing -> create `<project>/Tasks/<YY.MM> [<KEY>-123] <topic>` (reviewer: `<YY.MM> <KEY>-123 review <topic>`; topic from the ticket title) with `Status.md` from `status-template.md`, filled from what the branch shows; give the full path in the report, the developer may rename. Why: one place per task, design notes and agent state together.
6. Report in max 4 lines: ticket, role, step, exit condition, next action.

## Where records go

| Record | Place |
|---|---|
| Step, scope, decisions, follow-ups, PR reminders, reviewed sha | `Status.md` in the task folder (agent state, owned by the agent) |
| Concept drafts (`Concept.md`), questions, discussions, ticket splits | other files in the task folder (the developer's design notes), one topic per file; never delete or move them |
| Design decision with its trade-off | ADR (`adr`), linked from `Status.md` Decisions |

## Steps (author)

| # | Stage | Step | Tool | Exit condition |
|---|---|---|---|---|
| 1 | Requirements | Triage | `ticket-triage` | Risk + reason, hotspots; WHAT clear, open questions answered by their owner |
| 2 | Design | Concept | `adr` (look up), `concept`, `adr` (record) | Closed concept agreed; ADR written, Decisions links it. Only when HOW is unclear (triage) |
| 3 | Implementation | Implement | repo docs | Code + tests implement the concept, nothing more; a bug fix starts with a test that fails |
| 4 | Verification | Run | - | Build, unit tests and integration tests against every supported backend pass, the API called when behavior changed; output shown |
| 5 | Verification | Blind review | `branch-review` in a sub-agent | One run per branch, every risk level |
| 6 | Verification | Close review | `branch-review` filter | Each finding verified by a second agent or probe and reduced to the invariant it breaks; developer picked, fixed, step 4 re-run, review closed |
| 7 | Delivery | Commits | `git-recipes.md` | One commit per concern (only when asked) |
| 8 | Delivery | PR | `pr-template.md` | Draft shown; the developer posts it |
| 9 | Delivery | PR comments | `pr-comments` | Comments triaged, fixes done, reply drafts shown |
| - | Retrospective | Done | `retro` | Friction turned into proposals; a problem found late (PR review, test env, prod): which step should have caught it -> fix that step's skill. After merge (the ticket's PRs merged and none still open, e.g. `gh pr list --head <branch> --state all`): move the task folder to `Tasks/Archive/` and report it. Why: archiving a session doesn't end a task; the merge does |

## Steps (reviewer)

5 blind review (`branch-review`, colleague mode) -> 6 verify findings, the developer picks -> comment drafts in this session -> re-review: open threads + delta since the reviewed sha. Comments stay drafts; the developer posts them. `Status.md` keeps the reviewed sha and Follow-ups.

`git-recipes.md`, `pr-template.md` and `status-template.md` live next to this skill.

## While working

- Each step loops do -> check until its exit condition is shown. Not converging (each fix brings a new finding in the same area, > 3 rounds, the change outgrows the problem, tests bent to pass) -> stop, name the broken assumption and propose going back to the step that owns it (WHAT -> 1 / product owner, HOW -> 2), not just one step. The developer decides; record the back-jump in Decisions and, if it changes a decision, in the ADR. Why: patching downstream of a wrong assumption doesn't converge.
- Announce each step change in one line and update the Step line in `Status.md`. Record decisions, follow-ups and PR reminders as they happen.
- Off-step requests are fine: do them, name the step. Skipping a step: say why once; the developer decides.
- Ticket vs global problem: use the `branch-review` scope gate. Global -> own ticket first.
- Scope widening or reopening a closed review: point at the rule once, then follow the developer and record it.

## Step rules

- **3** High risk: before implementing, ask whether another developer checked the plan (`ticket-triage` rule); record the answer in Decisions. Why: no step asked for it, so a High ticket with a clear HOW went straight to code.
- **3** Test our code and our queries, not framework, SDK or library behavior. One test per distinct behavior; the same path with other inputs -> a parameterized test; not the same behavior in both unit and integration tests. Why: such tests cost upkeep and never fail because of us.
- **3** Before adding handling for a case the concept didn't list (an exception, state or edge found while building): name the app path that produces it (or "only corruption/manual edit"), and what leaving it costs. No reachable path or no real cost -> don't implement; mention it once as a follow-up or ask. A developer's "what about X?" asks whether X is covered, not for code. Why: once "every repeating exception -> Failed" was taken as the requirement, and handling for an unreachable state was built twice and fully reverted.
- **3** Before reporting done: run `concept` step 3 on what was built; report one line (what was dropped or simplified, or "nothing to drop" and why). Why: implementation exposes parts the concept didn't need; without this step, scope was re-checked only on request.
- **5** Prompt gets: skill path, repo + branch, range (without stacked commits), ticket text, Scope, Decisions and linked ADRs; not the author's opinions. Read-only; check `git status` afterwards. Sub-agent with the strongest available model and high reasoning effort, at every risk level (standing permission; other sub-agents only when the developer asks). Why every level: it costs about what a self-check in the authoring session costs, and only an independent context finds blind spots. High risk: the prompt adds a security pass (auth, roles, tenant isolation, personal data). Why: at High risk an auth, tenant or personal-data slip is the costliest kind; the review already has the context.
- **6** Verify each finding (second agent or probe) before rating it; present them in the `branch-review` §5 format plus a recommendation. `[Pre-existing]` -> Follow-ups. Closing the review and re-review of new commits: `branch-review` §5.
