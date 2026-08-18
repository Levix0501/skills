---
name: create-spec
description: Turn settled requirements and design decisions into an approved, frozen authority that a fresh implementation context can build and verify without conversation history. Use when the user asks to create a build-ready spec.
---

Set `WORKSPACE_ROOT` to the root opened for the current design and keep it fixed. Do not replace it with a Git repository root.

Write the spec to `<workspace-root>/docs/specs/YYYY-MM-DD-<topic>.md` from the current conversation context and workspace understanding, using [assets/spec-template.md](assets/spec-template.md). Git layout does not change this location. Do not re-interview the user — synthesize what the interview settled. The only questions still allowed are the product-level gaps defined below.

If `WORKSPACE_ROOT` is inside a Git worktree, that repository owns the spec's durability. If it is on main/master, first ask whether to open a branch for the spec. If `WORKSPACE_ROOT` is outside Git, warn once before writing that the spec has no Git-backed durability and may be lost; do not create a repository or relocate the document.

## The handoff contract

The approved spec is the sole requirements source: a fresh context holding only this document and the workspace must be able to implement and verify the work. Conversation history is not authoritative — anything the build needs must be in the spec.

No open product question survives to review — one gets asked now, in this window, never shipped as a TBD.

**User approval means frozen.** From approval onward the file is read-only for every process, this one included: no amendments, clarifications, status or build bookkeeping. When Git owns the spec, commit the approved file for durability; the commit records approval but does not create it. Outside Git, an explicit implementation request naming the file carries the approved handoff into a fresh context. The selected implementation workflow records progress and completion outside the spec.

A requirements change after approval is therefore a **new spec file**, written through this skill with the full review gate, carrying one extra metadata line:

```
Supersedes: <old-spec-path>
```

The old file is never edited — not to point at its replacement, not to mark it dead. The new file names it, and Git history preserves it when available. A build already running against the superseded spec is abandoned by its implementation workflow, not quietly re-aimed.

## Traceability

Requirements are numbered `R1..Rn`, each stating a clear obligation in observable terms. Acceptance criteria are numbered `A1..An`, each naming the requirement or requirements it verifies and the verification method — the exact command where one exists. Every requirement is covered by at least one acceptance criterion. These stable identifiers let the implementation workflow track coverage across phases without changing the spec.

## Length discipline

Write only what a fresh context needs to implement and verify the approved behavior. Use short prose, lists or tables as clarity demands; do not add filler or restate facts that the code already makes obvious. Do not include an execution task list or phase plan — the implementation workflow decides how to build the spec from the live code.

## Gaps found while writing

Four kinds, four responses:

- A codebase fact → look it up, never ask.
- An ordinary reversible implementation detail → leave it to the implementation run; do not put it in the spec.
- A reversible choice needed to make behavior, acceptance or the settled design unambiguous → decide it and record it under Decisions as `(author's choice)`.
- Anything that changes user-visible behavior, scope, acceptance, compatibility or an irreversible design commitment → ask the user now, in this window. Never invent a requirement to kill a TBD.

## Design discipline

Explore the current structure and follow established patterns. Record only architectural boundaries or targeted structural changes needed to deliver the approved behavior safely; leave ordinary implementation choices and unrelated refactors to the implementation run.

## Spec self-review

After writing, re-read the spec with fresh eyes — as the build window will read it, holding only this document and the workspace — then run the checklist in [references/self-review.md](references/self-review.md). Fix any issues inline — no need to re-review, just fix and move on.

## User review gate

After the self-review, ask the user to review the written spec before proceeding:

> "Spec written to `<path>`. Decisions I made while writing: <the `(author's choice)` list, or 'none'>. Please review it and let me know if you want to make any changes before we move on."

Wait for the user's response. If they request changes, make them and re-run the self-review. Once they approve, the spec is frozen; there is no status line to flip.

- Git-backed spec → commit only the approved spec. When it grew out of a tracked `docs/roadmap/` entry owned by the same repository, remove that entry in the same commit; nothing else rides along.
- Spec outside Git → do not commit, create a repository or move the file. Remind the user that approval has no durable record outside the current conversation, so a later implementation request must explicitly name and authorize the file. If it grew out of a roadmap entry, ask before removing that unversioned source.

Use `implement` for approved builds; it can coordinate whichever Git repositories the spec affects, including from a non-Git workspace.

Honor an already-stated build timing. Otherwise ask what happens next (AskUserQuestion), two options:

| Choice | Then |
|---|---|
| Build now | invoke `implement` with the approved spec's actual path |
| Later | stop and show `/implement <SPEC_FILE>` using that actual path; do not promise bare spec discovery |
