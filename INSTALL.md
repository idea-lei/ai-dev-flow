# Install

For people: give this file to your coding agent, e.g. "Install the skills from this repo per INSTALL.md." The rest is addressed to the agent.

---

You are installing a set of skills for your user. The user decides; you prepare, propose and show what changed. Change nothing outside this repo without the user's OK.

## Goal

The 7 skills in `skills/` (each folder with a `SKILL.md`) load in your user's agent, with the placeholders set and the defaults adapted to the user's system, without breaking what the user already has.

## What to do

1. **Place.** Find where your agent loads user-level skills and user-level instructions (or per-repo ones, if the user prefers). Copy the skill folders there with their names unchanged: they reference each other by name and by relative path (`../branch-review/...`). An existing skill with the same name -> ask before replacing.
2. **Instructions.** `skills/instruction.md` is not a skill. Merge it into the user's existing user-level instructions; on overlap or conflict, show both and let the user choose.
3. **Placeholders.** Set these in the copies (or in this repo before copying), never in the user's other files:
   - `<notes-root>`: private notes folder for task folders and ADRs, ideally a local git repo outside the code repo.
   - `<project>`: leave as is; the agent picks the folder per repo at run time.
   - `<KEY>`: the tracker's ticket prefix (`<KEY>-123` -> `PROJ-123`).
   - `develop`: the integration branch, where it names a branch (not inside words like "developer").
   - `<developer>`: the user's handle for the signature of AI-drafted replies.
   Detect what you can (default branch, ticket prefix in branch names, the code-host handle) and confirm with the user; ask only for the rest. All other angle-bracket placeholders (`<n>`, `<sha>`, `<topic>`, `<branch>`, ...) are filled at run time: leave them.
4. **Adapt.** Read the user's main repo and its conventions, then propose changes to these judgment sections; the defaults work, a match works better:
   - `branch-review/code-review.instructions.md`: the risk matrix (what "rarely", "probably" and "high consequence" mean at the user's scale and domain), "Do not suggest", "Check first".
   - `ticket-triage` hotspots: drop areas the system doesn't have, add the ones that hurt.
   - Assumptions that may not hold: integration tests against real backends, a separate infrastructure repo, ADRs kept outside the code repo, the agent drafts and the user posts.
   - Mechanical conventions belong in a linter or test, not in the skills (`retro` principle 5).
   Optional: suggest a `branch-review/known-facts.md` for accepted decisions that look like bugs.
   If the user's code host has a review bot that reads instruction files, offer to copy `code-review.instructions.md` into the repo's instructions folder so both reviewers follow the same rules.
5. **Verify.** Show the diff of everything you placed or merged. Then check that the skills load: on a ticket branch, "where are we?" should load `ai-dev-steps` and report ticket, role, step and next action in a few lines.
