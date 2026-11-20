---
name: adr
description: "Record or look up design decisions (ADRs). Use when a design choice is agreed (concept closed, approach changed after review), when asked 'write an ADR', 'record this decision', or 'why / what did we decide about X'."
---

# ADR

Folder: `<notes-root>/<project>/ADRs` holds the ADRs and the index (`README.md`). Private notes, never in the code repo. Why: trade-offs get forgotten after the ticket; the next design should start from them.

## Convention

- When: a concept was agreed, or a back-jump changed an earlier decision. Straight-path tickets get no ADR.
- File: `NNNN-short-title.md`, numbers never reused, from `adr-template.md` next to this skill.
- One screen. Options with their cons and the accepted trade-off are mandatory; the rest is optional.
- Once `Accepted`, the Decision is not rewritten: a changed decision is a new ADR and the old one gets `Superseded by NNNN`; small corrections and back-jumps go into `Revisions`.
- Status: `Proposed` -> `Accepted` -> `Superseded by NNNN` | `Rejected`.
- Tags: the area (`schema`, `messaging`, `auth`, `tenant`, `search`, `files`, `process`, ...), so the index can be filtered.
- No secrets, credentials or personal data.

## Look up

Before a concept, and when asked "why did we ...": filter the index by tag or ticket, then grep the files. Cite the ADR number; flag one that the current task contradicts.

## Record

1. Next free number; copy `adr-template.md`; fill from the agreed concept: Context, Options with cons, Decision with the accepted trade-off, Revisit when. One screen.
2. Back-jump that changed a decision: a small change -> a `Revisions` line; a different decision -> new ADR, old one `Superseded by NNNN`.
3. Add or update the index row in `README.md`.
4. Show the draft; write only after the developer's OK.
