# Spec template

The skeleton `create-spec` fills in. User approval freezes the spec. If Git
owns the file, `create-spec` commits it for durability; otherwise the user is
warned that it may be lost. The spec carries no status or build-lifecycle
field, and implementation never writes back to it.

```markdown
# <Topic> design

## 1. Goal / Non-goals

## 2. Requirements

### R1 — <short name>

<a clear obligation stated in observable terms>

### R2 — …

## 3. Context & constraints

<non-obvious current facts, affected systems or repositories, external
contracts and constraints; not an exhaustive file map>

## 4. Design

<settled architecture, boundaries and contracts; use
`None — implementation choice` when no design decision must be fixed here>

## 5. Compatibility, migration & failure behavior

<behavior and invariants that must survive, rollout or migration constraints,
and failure or edge-case behavior>

## 6. Acceptance

- A1 (R1): <observable end-state check — include the exact command and its
  working repository or directory where one exists>
- A2 (R1, R2): …

## 7. Decisions

<settled product or design decisions, plus any necessary choice made while
writing and marked `(author's choice)`; omit ordinary implementation choices>

## 8. Out of scope / rejected alternatives
```

All eight sections, in this order. Scale each to its complexity: one line if
trivial, `None — <why>` if not applicable. Never pad.

Requirement ids are `### R<n>` headings, exactly that shape so every
implementation workflow can map and verify them after the spec is frozen.
Acceptance criteria use `- A<n> (R...): <verification>` so their coverage is
explicit. Criteria may cover several requirements and should include
integrated or cross-system verification when the behavior requires it.

One metadata line appears only on a replacement spec:

| Line | Written by | Meaning |
|---|---|---|
| `Supersedes:` | create-spec | on a replacement spec only: `<old-spec-path>`. The superseded file is never edited |

There is no `Baseline:`, status or build-lifecycle line. The implementation
workflow reads the current code, freezes the approved content independently
and owns its own progress and completion records.
