# Scoped Re-Review Prompt Template

One fresh re-reviewer owns one complete fix-wave verdict.

**Purpose:** Decide whether every supplied finding is addressed or invalid, and report any regression caused by the fix wave.

```
Subagent (general-purpose):
  description: "Re-review [PHASE_ID], fix round [FIX_ROUND]"
  prompt: |
    Re-review fix round [FIX_ROUND] for phase [PHASE_ID].

    ## Contract

    Spec: [SPEC_FILE]

    Current scope:
    [SCOPE_BLOCK]

    Read both in full. The spec defines required behavior; the scope block defines the current phase boundary.

    ## Review wave

    Open findings:
    [FINDINGS]

    Controller-verified fix evidence and fixer concerns:
    [FIXER_STATUS]

    The finding lines are copied verbatim from the current review wave. Treat a fixer concern as a question to decide against the contract and code, not as a conclusion.

    ## Fix landing

    Repository fix ranges:
    [REPOSITORY_RANGES]

    Diff package: [DIFF_FILE]

    Use the package and any repository context needed to judge the fixes and their effects.

    ## Re-review

    Judge every supplied finding against the code at the fixed heads. Mark it addressed only when the specific defect no longer exists. If it remains, cite the current evidence. Mark it invalid only when the code and approved contract show that the original finding did not hold.

    Inspect the fix wave for regressions. A new finding must be caused by this fix wave, though its effect may appear in unchanged dependent code. Do not reopen unrelated pre-existing issues or future requirements.

    This work is read-only. Do not modify repositories, the spec or `.implement/`.

    ## Return

    Return only these two blocks. Give one verdict for every supplied finding, in order:

        VERDICTS:
        F1 | addressed
        F3 | not_addressed | repo/path.ts:42 still bypasses the required fallback
        F4 | invalid | the reported path is unreachable under the approved contract

        NEW_FINDINGS:
        F7 | important | repo/client.ts:18 the fix breaks callers without a token

    Begin new findings at [FIRST_FINDING_ID] and use the same `critical`, `important` and `minor` severities as the phase review. Critical means severe security, data-loss or irreversible production risk. Important means the current scope or stable landing cannot be trusted without a fix. Minor means a concrete non-blocking quality issue. Every new finding states the concrete failure or risk and cites the relevant repository path and line. Write `(none)` under `NEW_FINDINGS:` when the fix wave introduced no finding.
```

**Placeholders:**

- `[PHASE_ID]` — the phase whose landing is being re-reviewed.
- `[FIX_ROUND]` — the fix round being verified.
- `[SPEC_FILE]` — the approved spec.
- `[SCOPE_BLOCK]` — the same frozen current-scope contract used for implementation, review and fixing.
- `[FINDINGS]` — every open blocking finding given to the fixer, copied verbatim from the current review wave.
- `[FIXER_STATUS]` — the mechanically verified fixer status, including every concern.
- `[REPOSITORY_RANGES]` — one `<canonical repo path>: <fix-base>..<head>` line per fix repository.
- `[DIFF_FILE]` — the fresh fix package produced by `scripts/review-package`.
- `[FIRST_FINDING_ID]` — one past the highest finding id in the current review wave.

**Re-reviewer returns:** one verdict for every supplied finding and any new fix-caused findings. The controller applies the verdicts to the working wave and records only the remaining open blocking ids and deferred Minor one-liners.
