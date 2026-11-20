---
name: pr-comments
description: "Triage and address PR review comments - classify each thread, fix agreed ones locally, draft replies. Use when asked to 'check PR reviews', 'check the review comments', 'are the reviews reasonable', or 'fix the PR comments'."
---

# PR review comments

Read-only on the code host: never post, resolve or edit; reply drafts only, the developer posts them. No git changes; fixes are local only. Every reply draft ends with `_(AI-drafted, posted by @<developer>)_` (at the end, so a leading tag such as `noise:` stays first).

1. **PR:** current branch (`gh pr view --json number,url,headRefName,author`) or the given number. Branch not checked out -> read-only via `gh pr diff <n>`. Not the developer's PR -> triage only, the author fixes.
2. **Threads** (unresolved review threads with path, line and comments):
   ```text
   gh api graphql [--hostname <host>] -F owner=<owner> -F repo=<repo> -F pr=<n> \
     -f query='query($owner:String!,$repo:String!,$pr:Int!){repository(owner:$owner,name:$repo){pullRequest(number:$pr){reviewThreads(first:100){nodes{id isResolved isOutdated path line comments(first:20){nodes{databaseId author{login} body}}}}}}}' \
     --jq '.data.repository.pullRequest.reviewThreads.nodes | map(select(.isResolved|not))'
   ```
   Also `gh pr view <n> --json reviews,comments`. Outdated threads: check whether the code already addresses them. A pasted review text -> evaluate that instead.
3. **Triage** against the code and repo conventions, not generic advice (bots often suggest null guards or doc comments the conventions forbid). Rate per `../branch-review/code-review.instructions.md` (comment level + type, risk matrix). No own new findings.

   | # | Reviewer | Tag | File:Line | Comment | Verdict | Reason / change |

   Tag: the reviewer's (`[Important] bug` ...), or your own rating if untagged. Verdict: agree / partly / disagree / already done / follow-up (pre-existing or generic -> own ticket) / question. A `disagree` on a person's `[Important]` needs a reason they can accept, a talk or a small experiment. Not converging (`ai-dev-steps` "While working") -> suggest going back to Design (`concept`). Then wait.
4. **Apply** picked items, build + tests, show all reply drafts. Each draft starts with where to find it: `path:line` + the first words of the comment it answers (`-> "Consider disposing..."`). Why: the developer posts by hand and can't find threads by id.
   - Wrong AI comment (not real, or level too high): `noise: <reason>`. Skipped `[Nit]`: `skip`. `[Pre-existing]` / follow-up: the ticket key.
   - Say which threads to resolve: an `[Important]` one only when fixed or dismissed with a valid reason.
