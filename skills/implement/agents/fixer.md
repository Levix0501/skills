# Fix Wave Prompt Template

One fresh fixer owns one complete review wave.

**Purpose:** Resolve every open blocking finding coherently, verify the resulting landing, commit it and return a compact status block.

```
Subagent (general-purpose):
  description: "Fix [PHASE_ID], round [FIX_ROUND]"
  prompt: |
    Own fix round [FIX_ROUND] for phase [PHASE_ID] through completion.

    ## Contract

    Spec: [SPEC_FILE]

    Current scope:
    [SCOPE_BLOCK]

    Read both in full. The spec defines required behavior; the scope block defines the current phase boundary. Neither is editable.

    ## Findings

    [FINDINGS]

    These are the open blocking finding lines from the current phase review wave. Resolve every one.

    ## Landing

    Authorized repositories and fix bases:
    [REPOSITORIES]

    Reviewed change package: [DIFF_FILE]

    Use the findings, reviewed change and current code to understand each problem and fix its root cause. Make any supporting change needed for a coherent fix without expanding the approved scope.

    Follow the instructions and conventions governing the code you change. Work only in the authorized repositories. If complete resolution requires another repository, return BLOCKED before changing it so the controller can add it to the wave.

    Commit coherent changes, leave every named repository clean and run the tests, integration checks and affected acceptance verification needed to support the result. Do not edit the spec, phase brief or progress ledger; switch branches; push; or rewrite earlier history.

    If a finding does not hold or conflicts with the approved contract, do not force a code change. Name the finding and your reason in a `CONCERN` line so the re-reviewer can decide it.

    ## Return

    Return only one compact status block in your final message.

        STATUS: DONE
        REPO: <path> | COMMIT: <sha> | TESTS: <commands and results>
        INTEGRATION_TESTS: <commands and results, or n/a>
        ACCEPTANCE: <affected phase acceptance checks and results>

        STATUS: DONE_WITH_CONCERNS
        REPO: <path> | COMMIT: <sha> | TESTS: <commands and results>
        INTEGRATION_TESTS: <commands and results, or n/a>
        ACCEPTANCE: <affected phase acceptance checks and results>
        CONCERN: <finding id when applicable — one self-contained concern; repeat as needed>

        STATUS: BLOCKED
        REPO: <path> | COMMIT: <sha> | TESTS: <commands and results>
        BLOCKER: <what stopped the wave>
        REMAINING: <which findings remain and what is left>

    For a DONE variant, repeat `REPO:` for each changed repository and omit it when no code change was warranted. DONE means every finding was resolved. DONE_WITH_CONCERNS means all actionable work is complete but the re-reviewer must decide a stated concern or disputed finding.

    For BLOCKED, repeat `REPO:` for each repository containing committed fix work and omit it when none exists. Commit only coherent work and identify everything that remains.
```

**Placeholders:**

- `[PHASE_ID]` — the phase whose landing is being fixed.
- `[FIX_ROUND]` — the authorized round number.
- `[SPEC_FILE]` — the approved spec.
- `[SCOPE_BLOCK]` — the same frozen current-scope contract used for implementation and review.
- `[FINDINGS]` — every open blocking finding copied verbatim from the current review wave.
- `[REPOSITORIES]` — one line per authorized repository: `<canonical path> | branch <name> | fix base <sha>`.
- `[DIFF_FILE]` — the reviewed phase or prior fix package produced by `scripts/review-package`.

**Fixer returns:** committed fixes and one status block directly to the controller. The controller mechanically verifies the result; the fresh re-reviewer decides whether each finding is addressed or invalid.
