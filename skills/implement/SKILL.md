---
name: implement
description: Turn an approved spec into reviewed, independently shippable code with adaptive phase boundaries and reliable recovery across repositories. Use when the user asks to build an approved spec.
---

# Implement

You orchestrate this build and decide each next phase; separate fresh subagents implement each phase, review every landing, fix each review wave and re-review every fix.

## Flow

```text
Resolve spec
  → Open or create the progress ledger
  → Resume, or decide the current phase
  → Implement the phase
  → Review, fix and re-review until accepted
  → Choose the next phase or finish
```

## Rules throughout

- The controller owns `PROGRESS_FILE`. The approved spec and each dispatched phase brief are read-only; requirement changes go through `create-spec` in a new file carrying `Supersedes: <old-spec-path>`.
- Continue without routine check-ins. Stop only for a destructive or external action requiring user authority, an external or irreversible blocker, an approved spec that cannot be satisfied without changing its requirements, or the review user gate.

For each fresh dispatch, fill the named agent template's placeholders and use its `prompt` block as the subagent prompt.

## 1. Resolve spec

Stop at the first match:

1. The path passed with the command.
2. The spec written or approved earlier in this context.
3. Ask which spec to build.

## 2. Preconditions

1. Inspect every involved Git repository. For each, require a clean tree and ask whether to create a branch when on main/master.
2. Create or reuse the spec's implement directory:

   ```sh
   IMPLEMENT_DIR=$(scripts/implement-directory "$SPEC_FILE")
   PROGRESS_FILE="$IMPLEMENT_DIR/progress.md"
   ```

## 3. Progress

If `PROGRESS_FILE` exists, resume from section 9.

Otherwise create `PROGRESS_FILE` with:

```text
# Implement ledger — <SPEC_FILE>
```

The ledger is append-only and controller-owned. All ledger entries named in this skill are appended to `PROGRESS_FILE`. Record only decisions, evidence and checkpoints needed to resume or report the build. Use the spec's own identifiers when present; do not invent them.

Record a ruling only when a non-obvious decision changes scope, risk or recovery:

```text
Ruling: <decision> — <reason>
```

For each phase, choose one scope entry. A normal path needs only:

```text
Phase <n>: scope — whole remaining spec
Phase <n>: scope — brief: <PHASE_BRIEF>
Phase <n>: repositories — <repo-a> [<branch>@<base7>]; <repo-b> [<branch>@<base7>]
Phase <n>: implemented — <repo-a> <base7>..<head7>; <repo-b> <base7>..<head7> — verified: <test, integration and acceptance summary>
Phase <n>: accepted — <satisfied scope>
Build: complete — <repo-a> <first-base7>..<head7>; <repo-b> <first-base7>..<head7>
```

Append these only when they occur:

```text
Phase <n>: concern — <verbatim implementer concern>
Phase <n>: review — blocking
F<n> | critical|important | <finding>
Phase <n>: review: minor deferred — F<n>: <one-liner>
Phase <n>: fix round <R> repositories — <repo-a> [<branch>@<base7>]; <repo-b> [<branch>@<base7>]
Phase <n>: fix round <R> concern — <verbatim fixer concern>
Phase <n>: fix round <R> — <repo-a> <a7>..<b7>; <repo-b> <a7>..<b7> — verified: <test, integration and acceptance summary>
Phase <n>: re-review <R> — open: <finding ids>
Phase <n>: review: parked — F<n> — <user decision, reason and risk>
Build: abandoned — <reason>
```

## 4. Choose the current phase

Read the spec in full and inspect the code relevant to the remaining requirements. Determine what is satisfied and what remains from the current code and accepted phases.

Start with all remaining requirements in the tentative current phase.

Use the spec's own requirement and acceptance identifiers when available. Do not invent identifiers when they are absent.

Take a boundary only when it produces a material benefit:

- isolates a dominant uncertainty or high-risk seam that could invalidate later work;
- permits a migration, compatibility transition, external dependency or production observation to settle before continuing;
- separates substantially different systems, mental models or verification loops; or
- preserves enough headroom when the whole remaining spec no longer fits one reliable implementation run.

Every extra phase pays a phase tax: another context load, dispatch, review, fix loop and full-suite run. If the risk or cognitive-load reduction is smaller than that tax, cross the boundary and keep the work together.

If no boundary is worth that tax, append the scope decision and dispatch the whole remaining spec directly. Generate no phase brief:

```text
Phase <n>: scope — whole remaining spec
```

If a boundary is worth taking, choose the largest coherent stable increment on its near side. Include its full dependency closure. Its landing must satisfy:

- tests pass and every affected repository is safe to merge or deploy;
- current behavior remains compatible unless the spec explicitly changes it;
- no temporary stub, broken path or half-migration needs a later phase to become safe;
- later requirements can add to this state rather than replace or undo it;
- the result has observable acceptance evidence.

Absorb all low-risk requirements that share the same code context, risk model and verification path, including multiple independently testable outcomes. Stop only when adding more work crosses the identified worthwhile boundary or removes the headroom needed for a reliable run.

Split on valuable risk and cognition seams, not on technical layers or repository boundaries. One behavior spanning several repositories is normally one phase unless a compatibility-safe rollout creates a worthwhile stable boundary. Merge candidates when they share implementation context or tests, are individually cheap relative to the phase tax, share an invariant or transaction, one is unsafe or useless without the other, or the next phase would have to reload and rewrite the first.

Use the spec and current phase to identify every Git repository whose tracked state must change for the landing. Do not assume or require a particular repository layout. Record each repository by its canonical path.

For a stable increment, set `PHASE_BRIEF="$IMPLEMENT_DIR/P<n>.md"` and write only the current phase brief:

```md
# P<n> — <observable outcome>

Requirements: <the spec requirements included in this phase, preserving their identifiers when present>

Landing: <the independently deployable end state and exact stop boundary>

Verify: <the spec acceptance criteria, preserving their identifiers when present, and only phase-specific checks>
```

Append `Phase <n>: scope — brief: <PHASE_BRIEF>`. The brief narrows only the current scope; the spec remains authoritative. Freeze it at dispatch so implementation, recovery and review use the same contract.

## 5. Phase loop

For each repository in the current phase, record its canonical path, current branch and current HEAD as that repository's phase base:

```sh
git -C <repo> rev-parse HEAD
```

Capture every phase base before implementation starts, then dispatch one fresh implementer for the whole phase using [agents/implementer.md](agents/implementer.md):

- whole-remaining scope → pass no phase brief; direct it to implement every spec requirement not already accepted;
- stable-increment scope → pass the frozen `PHASE_BRIEF` together with the spec.

The implementer owns the complete phase across all named repositories, including integration, tests and acceptance verification. It may make several coherent commits in each changed repository, but the final commit in every changed repository must carry `Phase: P<n>`.

It returns one of these status blocks directly to the controller and writes no implementation report.

For DONE, repeat `REPO` for each changed repository:

```text
STATUS: DONE
REPO: <canonical path> | COMMIT: <sha> | TESTS: <commands and results>
INTEGRATION_TESTS: <commands and results, or n/a>
ACCEPTANCE: <phase acceptance checks and results>
```

For DONE_WITH_CONCERNS, repeat `REPO` and `CONCERN` as needed:

```text
STATUS: DONE_WITH_CONCERNS
REPO: <canonical path> | COMMIT: <sha> | TESTS: <commands and results>
INTEGRATION_TESTS: <commands and results, or n/a>
ACCEPTANCE: <phase acceptance checks and results>
CONCERN: <one self-contained line>
```

DONE_WITH_CONCERNS must meet the same completion standard as DONE. Incomplete or unsafe work is BLOCKED.

For BLOCKED, repeat `REPO` for each repository containing committed phase work; omit it when none exists:

```text
STATUS: BLOCKED
REPO: <canonical path> | COMMIT: <sha> | TESTS: <commands and results>
BLOCKER: context|ambiguity|external|other
REMAINING: <what behavior, integration and tests remain>
```

For a DONE variant, mechanically verify before dispatching the reviewer:

- every repository expected to change has one `REPO` line;
- every returned commit equals its repository HEAD, descends from that repository's phase base and carries `Phase: P<n>`;
- every named repository tree is clean unless rescuing finished work verbatim;
- the spec, `PROGRESS_FILE` and `PHASE_BRIEF` when present are unchanged;
- every reported repository test, integration check and phase acceptance check passes when rerun at the final heads;
- the review ranges are exactly each repository's recorded phase base through its verified HEAD.

Append each returned concern, then one `Phase <n>: implemented — <repo> <base7>..<head7>; ... — verified: <test, integration and acceptance summary>` line covering every phase repository.

If the return is malformed or conflicts with repository state, inspect the repositories and reconstruct what can be verified. If the landing is incomplete, resume the same implementer; if it is fully implemented, proceed using the verified evidence. Do not accept an unfinished phase as a completed checkpoint.

For BLOCKED, inspect the committed work in every named repository rather than trusting the label. Resume the same implementer when the remaining work still belongs to the current stable landing. Resolve reversible ambiguity with a ruling; escalate external or irreversible blockers.

If implementation discovers another required repository, add it to the phase, record its branch and phase base, then resume the same implementer with the expanded repository set. The phase is not implemented until that repository is integrated and verified.

If no implementation can satisfy the approved spec without changing its requirements, append the ruling and `Build: abandoned — <reason>`, report and stop. Keep the implement directory as the terminal record.

## 6. Review, fix and re-review

After mechanically verifying the phase, build one package with `scripts/review-package "$SPEC_FILE" <repo> <base> <head> [<repo> <base> <head> ...]`. Dispatch a fresh reviewer using [agents/reviewer.md](agents/reviewer.md) with the spec, current scope, repository ranges, implementer concerns, test results and acceptance evidence. For whole-remaining scope, include the accepted phase records excluded from the current review in the scope block. The reviewer owns the complete review: requirement coverage, design alignment, stable landing, cross-repository integration and technical quality.

Keep the reviewer's finding block as the working review wave. Record each Minor as `Phase <n>: review: minor deferred — F<n>: <one-liner>`. If Critical or Important findings exist, append `Phase <n>: review — blocking` followed by those finding lines verbatim. Otherwise append `Phase <n>: accepted — <satisfied scope>` from the review result, preserving the spec's identifiers when present, then return to section 4 with the actual repository heads. Determining satisfied and remaining requirements is part of choosing the next phase, not a separate review or findings pass.

A blocking finding authorizes fix round 1 without a user decision. Append `Phase <n>: fix round <R> repositories — <repo> [<branch>@<base7>]; ...` for every repository the fixes may change. Dispatch a fresh fixer using [agents/fixer.md](agents/fixer.md) with the spec, current scope, authorized repositories and fix bases, the reviewed change package and every open blocking finding copied verbatim.

For a DONE variant, mechanically verify the fix before re-review:

- every returned commit equals its repository HEAD and descends from its recorded fix base;
- every named tree is clean;
- the reported repository tests, integration checks and affected phase acceptance checks pass when rerun;
- every fix range is exactly its recorded base through the verified HEAD.

For BLOCKED, inspect the committed fix work. Resume the same fixer when the remaining work still belongs to the authorized finding wave; escalate external or irreversible blockers. Do not re-review an incomplete fix wave.

After a mechanically verified DONE variant, append any fixer concerns and one `Phase <n>: fix round <R> — <repo> <base7>..<head7>; ... — verified: <test, integration and acceptance summary>` line. Build a fresh package covering those ranges and dispatch exactly one fresh scoped re-reviewer using [agents/re-reviewer.md](agents/re-reviewer.md) with the spec, current scope, open findings, verified fixer status and concerns, fix ranges, fresh package and next finding id. Apply its verdicts to the working wave and add new breakage. Record new Minor findings in the deferred form above. If blocking findings remain, append `Phase <n>: re-review <R> — open: <finding ids>` and append only newly introduced blocking finding lines verbatim; existing finding lines are already in the ledger.

Round 1 is the only automatically authorized fix round. When no blocking finding remains, record the phase as accepted from the review and re-review evidence. Only then choose the next phase from the current code.

## 7. User gate

If blocking findings remain, resolve them before choosing the next phase:

| Choice | Record and continue |
|---|---|
| Fix again | `Ruling: user authorized fix round <R> — <reason when given>`; return to section 6 for one fix and one re-review |
| Accept risk | `Phase <n>: review: parked — F<n> — Ruling: accepted by user — <reason and risk>` |
| Defer | `Phase <n>: review: parked — F<n> — Ruling: deferred by user — <reason and cost>` |
| Stop | append an abandonment ruling and `Build: abandoned — <reason>`, then report and stop |

Only the user accepts or defers Critical/Important findings, authorizes another round, or abandons for review reasons. The controller may defer a Minor finding.

A Critical or Important finding may be parked only when the accepted scope and stable landing remain true. A finding that shows a spec requirement is unsatisfied or the landing is unsafe requires a fix, a stopped build or a superseding spec.

When every blocking finding is fixed or validly parked, record the phase as accepted and return to section 4.

## 8. Finish

The build finishes on spec coverage, not on a predicted phase count. Confirm:

- every spec requirement is covered by accepted scope and review evidence;
- every phase has accepted review evidence for its stable landing;
- no open blocking finding, or each residual finding is parked with the required user ruling;
- every participating repository tree is clean;
- the accepted review history covers the current HEAD of every participating repository.

If any range is unreviewed, do not finish. Review every unreviewed range and resolve its findings through the same review, fix and re-review gate.

Collect the first base and current head of every participating repository and append one `Build: complete — <repo ranges>` line. Then collect each repository's branch and range, implemented outcomes, tests, acceptance evidence, deferred Minor findings, parked blocking findings and rulings. Delete this spec's implement directory and report the collected result. The implement directory is transient; repository histories are the durable implementation record, and the spec's durability depends on whether it is Git-backed. Integration is the user's next decision.

## 9. Resume

Read `PROGRESS_FILE` and verify that its first line names `SPEC_FILE`, then verify every recorded commit in its named repository. If a commit is reachable only from another branch, name that branch and ask before continuing. Inspect status and log for every named repository, then trust accepted phase lines and named commits over conversation memory. A terminal ledger never resumes implementation automatically.

- Stable-increment scope recorded but `PHASE_BRIEF` missing → choose the current phase again from the actual code and accepted phases.
- Current phase without a complete implemented line → inspect status, log and `Phase: P<n>` commits in every named repository. Apply the section 5 mechanical gate; if the landing is fully implemented, append its implemented line with the verification summary. Otherwise resume the same implementer when available, or dispatch a fresh implementer with the frozen scope and verified repository state.
- Implemented line present with neither an accepted line nor a blocking review → repeat the full phase review.
- A blocking review has no fix round → run the automatically authorized fix round 1.
- A user authorization has no later fix round → run that authorized wave.
- Fix-round repositories recorded but no completed fix-round line → inspect the fixer commits and apply the section 6 BLOCKED handling. Resume the same fixer if incomplete; otherwise mechanically verify it and append the completed fix-round line.
- A completed fix-round line has no later re-review or accepted line → re-review its recorded ranges.
- The latest re-review names open blocking findings → return to the user gate.
- Phase accepted with requirements remaining → choose the next phase again from the actual repository heads.
- `Build: complete` exists but the implement directory remains → collect the recorded result, delete only that directory and do not repeat completion work.
- `Build: abandoned` exists → report the recorded terminal state and keep the implement directory.

If `PROGRESS_FILE` or the implement directory is lost, do not infer accepted phases from commit trailers alone. Inspect the current code and repository history against the spec, repeat verification and review for uncertain spec-related work, and only then use the verified repository heads as the recovery baseline for choosing the next phase.
