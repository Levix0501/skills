# Phase Landing Reviewer Prompt Template

One fresh reviewer owns the complete review of one mechanically verified phase landing.

**Purpose:** Decide whether the phase delivers the right outcome and is well built, then return only concrete findings that can drive the fix gate.

```
Subagent (general-purpose):
  description: "Review phase [PHASE_ID]"
  run_in_background: false
  prompt: |
    Review one mechanically verified phase landing and return only concrete findings that can drive the fix gate.

    ## Contract

    Spec: [SPEC_FILE]

    Current scope:
    [SCOPE_BLOCK]

    Read both in full. The spec defines required behavior; a phase brief defines the current boundary. For a whole-remaining phase, the scope block names previously accepted scope; every other spec requirement is current. Report a material conflict rather than silently choosing or expanding the boundary.

    ## Evidence

    Repository ranges:
    [REPOSITORY_RANGES]

    Diff package: [DIFF_FILE]

    Implementer concerns, repository tests, cross-repository checks and phase acceptance results:
    [AGGREGATED_BLOCK]

    Use the package, reported evidence and any repository context needed to judge the complete landing at the named heads. Treat the reported verification as evidence to check, not as a conclusion.

    ## Review

    Judge both whether the landing fully satisfies the current contract and whether its engineering is trustworthy; neither can offset a defect in the other.

    Repository instructions and documented standards override generic preferences. An undocumented smell is a judgement call and is blocking only when it creates a concrete material risk. Report only defects introduced by or preventing this phase; exclude unrelated pre-existing issues, future requirements and speculative improvements.

    Verify every requirement in the current scope against code and observable evidence. Preserve the spec's identifier when it has one; otherwise cite its section or acceptance criterion. A standards finding cites the governing source and rule. Every finding names the relevant repository path and line and states the concrete failure or risk.

    This review is read-only. Inspect code or run verification as needed, but leave repositories, the spec and `.implement/` unchanged.

    ## Return

    Return only the finding block:

        F1 | important | scope: R2 — repo/path.ts:42 omits the required fallback when the service is unavailable
        F2 | minor | quality: CONTRIBUTING.md error-handling rule — repo/client.ts:18 discards the original error

    Begin at [FIRST_FINDING_ID], number consecutively and report one self-contained defect per line. Use exactly `critical`, `important` or `minor`; do not put another `|` in the finding text. A clean review returns `(none)`.

    Critical means severe security, data-loss or irreversible production risk. Important means the current scope or stable landing cannot be trusted without a fix. Minor means a concrete non-blocking quality issue. A preference or smell is blocking only when it violates a documented rule or creates a concrete material risk.
```

**Placeholders:**

- `[PHASE_ID]` — the current phase.
- `[SPEC_FILE]` — the approved spec.
- `[SCOPE_BLOCK]` — `Phase brief: <PHASE_BRIEF>` naming the frozen brief file to read, or the whole-remaining instruction together with the accepted phase records excluded from current scope.
- `[REPOSITORY_RANGES]` — one `<canonical repo path>: <base>..<head>` line per phase repository.
- `[DIFF_FILE]` — the combined package path printed by `scripts/review-package`.
- `[AGGREGATED_BLOCK]` — implementer concerns, repository tests, cross-repository checks and phase acceptance results.
- `[FIRST_FINDING_ID]` — one past the highest finding id recorded in the ledger; `F1` when none exists.

**Reviewer returns:** only the phase finding block. The controller keeps it as the review wave, records blocking finding lines and deferred Minor one-liners in the ledger, and records no finding block for a clean review.
