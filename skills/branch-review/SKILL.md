---
name: branch-review
description: "Review a branch or PR for bugs, convention violations, missing migrations/tests and weird code, without checking it out. Use when asked to 'check this branch', 'find problems', 'anything weird?', 'blind review', for a pre-PR check, or to review a colleague's PR."
---

# Branch review

Read-only: no git changes, no fixes until the developer picks findings. A colleague's branch: findings only, the author fixes.

## 1. Scope

- Blind review: fresh session or sub-agent, never the one that wrote the code. Facts only: ticket, Concept, conventions, diff; not the author's chat. Why: the author can't see its own blind spots.
- Current branch: `git --no-pager diff origin/develop...HEAD` + uncommitted changes. Other branch: `origin/develop...origin/<branch>`. PR: `gh pr view <n> --json title,body,headRefName,headRefOid,files` + `gh pr diff <n>`.
- `origin/develop` stale -> ask before `git fetch`. Stacked branch -> review only the delta to the parent.
- Check the diff against the ticket. Work split across tickets, stacked PRs or the infrastructure repo is not a finding; ask if unclear.
- Area facts (accepted decisions that look like bugs): a `known-facts.md` next to this skill, if you keep one.

## 2. Filters (apply first)

**Rating.** Comment level, type, risk matrix, "Do not suggest" and the high-risk areas: `code-review.instructions.md` next to this skill. Read it first.

**Scope gate.** In scope = introduced by this diff or required by the ticket. Global = shared code, test infra, conventions, pre-existing code. Global and needed -> own ticket first, this one stacked. Global and not needed -> Follow-ups. Unsure -> global. Out of scope -> level `[Pre-existing]`; goes to Follow-ups, never blocks. Why: global fixes inside a ticket make reviews endless.

**Worth it?**
- Asked to fix something: first check whether it's worth it. Fixes should fit existing types; new abstractions -> say so, rate `Nit`.
- Trace each impact end to end before rating it, then reason from first principles: which invariant or goal breaks, and why it must hold here. A matching pattern or rule is not enough; check the rule's reason against this case.
- Check shared code and the repo docs before calling something missing: response conventions, base classes, global filters and locking helpers often already cover it.
- "No problems" is a valid result.

## 3. Checklist

**Ticket:** stated risk fits what the diff touches; tests cover each acceptance criterion; bug fix -> root cause fixed and a test that failed before.

**High-risk areas:** "Check first" in `code-review.instructions.md`, every occurrence.

**Conventions:** repo instructions and development docs. Check the mechanical ones (naming of routes, messages, headers; enum values) where no linter or test does; better, move them into one (`retro` principle 5).

**Tests:**
- The repo's naming and structure (e.g. `Method_Scenario_ExpectedResult`, Arrange/Act/Assert).
- Database/HTTP behavior -> integration tests against every supported backend, through the real entry point.
- Shared test-data builders over ad-hoc helpers, if the repo has them; only the builder methods that are used. Helpers that do database/HTTP work or build raw bytes stay.
- Unnecessary: tests of framework/library behavior ("Do not suggest"), cancellation tests, guard throws, exact duplicates, the same behavior in both unit and integration tests. An integration test that asserts the HTTP status stays.

**Taste (max 3, first round only):**
- One commit per concern; feature and refactor not mixed.
- Public methods read top-down (guards -> transaction -> effect -> event), max 2 nesting levels.
- Names carry intent; one term per concept across API, model, storage and tests.
- New code looks like its neighbors.
- No hidden contracts on caller input; symmetric operations look symmetric.
- Removals complete; no speculative parameters; diff < ~800 added lines (excluding generated files).

**Hygiene:** no debug code, TODOs, dead code, unrelated changes or unjustified packages. Docs updated. Safety-relevant logic flagged for human review.

## 4. Verify

- Build + affected unit tests, only when the reviewed branch is checked out; otherwise the PR's CI (`gh pr checks <n>`) or "not run". Ask before integration tests. Why: a build of another checkout says nothing about the reviewed code.
- Optional: a second review agent in parallel; merge its findings.
- Perf/leak questions: measure with a temporary probe test, then delete it and check `git status`.
- Reverting own changes: no `git checkout`/stash; restore from `git show HEAD:<path>` only files you alone changed.

## 5. Output (ASCII)

1. **Summary:** 3-6 bullets.
2. **Findings** (in scope, `Important` first): overview `| # | Level | Type | File:Line | Problem |`, then per finding: snippet, why (concrete harm; matrix cell for `risk`), how (fix + size), how to verify, own take.
3. **Follow-ups** (`[Pre-existing]`): one line each.
4. **Tests:** only missing, unnecessary or broken tests, with reason.
5. **Build/tests:** one line.

Own branch: ask which findings to fix, fix locally only.

Closing (both modes): after the picked fixes the review is closed. No new full review; new commits -> review only that delta; only an in-scope `[Important] bug` reopens a full review. Why: AI findings can be wrong, and a review that never closes keeps producing work.

Colleague's branch or PR: ask which findings to raise, then draft one comment per finding in the `code-review.instructions.md` format (tag first; `[Important]` with Goal / Why / How / How to verify), with file:line. Drafts only; the developer posts them. Note the reviewed sha (`headRefOid`). Re-review after the author's changes: only the open threads and the delta since that sha.
