---
applyTo: "**"
excludeAgent: "cloud-agent"
---

# Code review rules

Goal: find real bugs, make little noise. A person owns the review and decides; your comments are input. Judge against this repository's conventions (its instructions and development docs), not generic advice.

## Comment format

Start every comment with one comment-level tag and one type: `[Important] bug: ...`, `[Nit] taste: ...`, `[Pre-existing] risk: ...`.

Comment level (does it block the merge?):

- `[Important]` - blocks the merge. Write it as:
  - **Goal:** what must hold.
  - **Why:** what breaks and how (for a bug: the cause).
  - **How:** the smallest fix that fits existing types and methods.
  - **How to verify:** the test or check that proves the fix.
- `[Nit]` - optional improvement. One or two lines, specific.
- `[Pre-existing]` - when this PR did not introduce the problem, whatever its impact. Suggest a separate ticket; it does not block the merge.

Type (what kind of problem?) and its default comment level:

| Type | Meaning | Comment level |
| --- | --- | --- |
| `bug` | Proven break, traced end to end | `[Important]` |
| `risk` | Breaks only under some condition (concurrency, failure path, load) | From the risk matrix below |
| `convention` | Breaks a repo rule | `[Important]` only when the rule protects correctness (migrations for every provider, tenant isolation, enum values never renumbered); else `[Nit]` |
| `taste` | Readability, naming, structure | `[Nit]`, at most 3, first review round only |

Risk matrix (calibrate "rarely" and "probably" to your system, e.g. one tenant per deployment, thousands of users, a few concurrent editors):

|  | rarely | probably |
| --- | --- | --- |
| low consequence | no comment | `[Nit]`, only if the fix fits existing patterns |
| high consequence | `[Important]` if the fix is cheap, else suggest a ticket | `[Important]` |

High consequence = silent, lasting damage (corrupted core data, cross-tenant leaks, safety). Low = visible or cosmetic. Self-healing -> one level lower. Name the cell in the comment.

Ticket risk (Low / Medium / High in the PR description) is a different scale: it rates the whole change, not one comment.

## Filter before commenting

- Comment only when the problem is real, useful and proportionate. Trace the claimed impact end to end before writing; if you can't show how it breaks, don't post it as `[Important]`. Name the invariant or goal it breaks; "looks like a known pattern" is not a finding.
- No over-engineering: no fix that needs new abstractions, models or interface overloads for a micro-gain. Say so explicitly if a fix needs them.
- Don't repeat what CI or the analyzers already report.
- "No findings" is a valid outcome. No praise, no summaries of what the PR does.

## Do not suggest (forbidden by our conventions)

List what your conventions forbid, so reviewers stop suggesting it. Ours:

- Defensive null checks where the type system already rules out null, or tests that pass null to force them.
- Doc comments or explanatory comments by default. A comment the PR adds is fine when it says *why* (a constraint, a workaround, a non-obvious reason); one that restates the code -> `[Nit]`.
- Tests of framework or library behavior (cancellation, request binding, plain ORM behavior, etc).

## Check first (high-risk areas)

- **Resources:** every disposable resource (streams, native handles, scopes) released on all paths: success, early return on failure, exceptions, cancellation, partially built collections. Clear ownership when a resource is passed on or wrapped; cleanup must not double-release or swallow the original exception.
- **Read-then-write:** a read + existence or state check followed by update or delete races with concurrent writers. Rate it with the risk matrix; if it is a finding, the write itself must decide (conditional update + affected rows, version token, constraint violation).
- **Tenant isolation and auth:** raw SQL, bulk updates/deletes, anything that bypasses global query filters, new endpoints without the right role. Any change to auth, permissions, tenant or personal data -> say whether a security look is needed.
- **Schema:** migrations for every supported database, consistent across providers, backward-compatible, data migration for existing rows; provider-neutral SQL where it is shared.
- **Messaging:** subscribers idempotent and tolerant of out-of-order delivery; side effects not skipped on retry; writes and publishes in the same transaction (outbox); no cascading delete where messages must be published. How depends on ownership: a service's own data -> refresh from the source (re-read by id, missing -> delete); replicas of another service's data -> version guard. Version or tombstone machinery on own data is over-engineering.
- **Contracts:** changes to shared DTOs, messages or enums affect other services; enum values explicit and never renumbered.
- **Regulated or safety-relevant logic:** flag it for explicit human review.

## Tests and evidence

- Do the tests cover the acceptance criteria stated in the PR? Name the missing case.
- Any redundant / unnecessary test cases that can be merged or removed?
- Bug fix: does it fix the root cause, and is there a test that failed before?
- Database, transaction or HTTP behavior -> integration test against every supported backend.
- The PR states a ticket risk with a reason; if the change touches the areas above and the risk looks too low, say so once.
