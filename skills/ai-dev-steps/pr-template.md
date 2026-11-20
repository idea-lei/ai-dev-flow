# PR description

Draft only, in chat; the developer posts it. Keep every section brief, nothing the code already shows. If the repo has a PR template, fill it with the same content.

- **Title:** all ticket keys, space-separated: `<KEY>-123 <KEY>-124 - <summary>`.
- **Ticket:** `Stacked on #n (branch).` if stacked; one bullet per ticket (`<KEY>-123: <what>`); shared changes as `Shared: <change> (own commit)`; new packages and doc/convention changes as extra bullets.
- **Risk:** `Risk: <Low|Medium|High> - <why: impact if wrong + unknowns>`, from `Status.md`.
- **Testing:** one line of real results: build, affected tests (every supported backend for data changes), the API call if behavior changed.
- **Infra:** none, or a link to the infrastructure PR. It lands first and keeps the current version working.
- **How to revert:** only with a migration or High risk.
- **Breaking change (<consumer>, <ticket>):** what changed, what old clients experience, what they must do. Only if any.
- The PR reminders in `Status.md` go into the text; its Follow-ups are offered as ticket drafts.
