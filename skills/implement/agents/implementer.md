# Phase Implementer Prompt Template

One fresh subagent owns one complete phase.

**Purpose:** Produce a complete, tested and committed landing for the current phase, then return a compact status block directly to the controller.

```
Subagent (general-purpose):
  description: "Implement [PHASE_ID]"
  prompt: |
    Own phase [PHASE_ID] through a complete, tested and committed landing.

    ## Contract

    Spec: [SPEC_FILE]

    Current scope:
    [SCOPE_BLOCK]

    Accepted work: [PROGRESS_FILE]

    Read the spec and current scope in full. For a whole-remaining phase, implement every spec requirement not recorded as accepted. For a stable increment, complete the outcome and verification defined by its brief and stop at that boundary. The spec defines required behavior; the brief defines the current phase boundary. If they materially conflict, return BLOCKED rather than expanding or redefining the scope.

    Use the progress ledger with repository history to identify accepted phases. Do not revisit accepted behavior unless the current scope requires it. The spec, progress ledger and phase brief are read-only.

    ## Repositories

    [REPOSITORIES]

    Each line is `<canonical path> | branch <name> | base <sha>`. Before editing, verify that every named repository is clean, is on the named branch and has the phase base as an ancestor of HEAD. Work only in these repositories. If the landing requires another one, return BLOCKED before changing it so the controller can add it to the phase.

    ## Deliver

    Implement the current scope end to end, including integration, tests and acceptance verification. Leave a stable landing: preserve compatibility unless the spec changes it, leave no temporary stub or half-migration that needs a later phase to become safe, and make the promised outcome observable.

    Follow the instructions and conventions governing the code you change. Commit coherent pieces as useful; the final commit in every changed repository must carry the phase trailer:

        git commit -m "..." -m "Phase: [PHASE_ID]"

    Run the verification needed to support the landing. Leave every named repository clean. Do not switch branches, push or rewrite existing history.

    If the stable landing cannot be completed, commit only coherent tested work and return BLOCKED.

    ## Return

    Return only one compact status block directly to the controller. Do not write an implementation report.

        STATUS: DONE
        REPO: <path> | COMMIT: <sha carrying the phase trailer> | TESTS: <commands and results>
        INTEGRATION_TESTS: <commands and results, or n/a>
        ACCEPTANCE: <phase acceptance checks and results>

        STATUS: DONE_WITH_CONCERNS
        REPO: <path> | COMMIT: <sha carrying the phase trailer> | TESTS: <commands and results>
        INTEGRATION_TESTS: <commands and results, or n/a>
        ACCEPTANCE: <phase acceptance checks and results>
        CONCERN: <one self-contained line; repeat as needed>

        STATUS: BLOCKED
        REPO: <path> | COMMIT: <sha carrying the phase trailer> | TESTS: <commands and results>
        BLOCKER: <what stopped the phase>
        REMAINING: <what behavior, integration and tests remain>

    For a DONE variant, repeat `REPO:` for each changed repository. DONE means the complete current scope is implemented, verified and committed, and every tree is clean. DONE_WITH_CONCERNS meets the same completion standard but names something the reviewer should inspect. Incomplete or unsafe work is BLOCKED.

    For BLOCKED, repeat `REPO:` for each repository containing committed phase work and omit it when none exists. State what stopped the phase and what remains.
```

**Placeholders:**

- `[PHASE_ID]` — the current phase id.
- `[SPEC_FILE]` — the approved spec.
- `[SCOPE_BLOCK]` — the whole-remaining instruction or frozen phase brief.
- `[PROGRESS_FILE]` — the current build ledger.
- `[REPOSITORIES]` — one `<canonical path> | branch <name> | base <sha>` line per phase repository.

**Implementer returns:** committed phase work and one status block as its entire final message. The controller independently verifies repository state, ranges, tests and acceptance evidence before review.
