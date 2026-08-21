---
name: implement
description: Turn an approved spec into reviewed, independently shippable code with adaptive phase boundaries and reliable recovery across repositories. Use when the user asks to build an approved spec.
---

# Implement

Orchestrate the build from an approved spec. Fresh roles implement each landing, review it, fix its finding wave, and re-review fixes. The controller owns scope, durable state, mechanical gates, and user decisions.

## Flow

```text
Resolve spec
  → Open or recover the ledger
  → Choose an intermediate or final phase
  → Dispatch by role path + immutable manifest
  → Verify agent-owned evidence
  → Review, fix and re-review
  → Carry eligible Minor findings or close the whole spec
  → Finish or choose the next phase
```

## Rules throughout

- The controller alone writes `PROGRESS_FILE` and dispatch manifests. The approved spec and frozen phase brief are read-only; requirement changes require a new approved spec carrying `Supersedes: <old-spec-path>`.
- Keep dynamic context in workspace artifacts. Agent prompts contain only the role-contract path, dispatch-manifest path, and compact return instruction; never inline spec text, evidence, project warnings, or finding lines when dispatching.
- Validate every manifest and evidence file before consuming it. Implementer and fixer may write only their manifest-named evidence path inside `IMPLEMENT_DIR`; reviewer and re-reviewer are read-only.
- Continue without routine check-ins. Stop only for destructive or external authority, an irreversible blocker, a contract that cannot be satisfied, or the review user gate.

For every fresh role, use the matching contract in `agents/`. The dispatch prompt is:

```text
Read and follow the role contract:
<role-contract-path>

Execute this dispatch:
<dispatch-manifest-path>

Return only the format required by the role contract.
```

Keep this prompt below 1 KB. Each role reads every authoritative input named by its manifest.

## 1. Resolve spec

Use the first available source:

1. the path supplied by the user;
2. the approved spec from the current context;
3. ask which spec to build.

## 2. Preconditions

Inspect every involved Git repository. Require a clean tree and ask whether to create a branch when on main/master. If a required repository does not exist, create it with an empty root commit; that commit is its first build base.

Create or reuse the run directory:

```sh
IMPLEMENT_DIR=$(scripts/implement-directory "$SPEC_FILE")
PROGRESS_FILE="$IMPLEMENT_DIR/progress.md"
mkdir -p "$IMPLEMENT_DIR/dispatch" "$IMPLEMENT_DIR/evidence"
```

`IMPLEMENT_DIR` is transient active-build state. The spec and repository history remain the authorities after terminal cleanup.

## 3. Progress ledger

If `PROGRESS_FILE` exists, resume from section 9. Otherwise create:

```text
# Implement ledger — <SPEC_FILE>
```

The ledger is append-only and controller-owned. Correct a defective entry by appending a ruling that names it; never rewrite or delete recorded state. Append only forms named here. Preserve spec identifiers when present and never invent them.

Record non-obvious scope, risk, authority, or recovery decisions as:

```text
Ruling: <decision> — <reason>
```

Normal entries:

```text
Phase <n>: scope — intermediate — brief: <PHASE_BRIEF>
Phase <n>: scope — final — whole remaining spec
Phase <n>: scope — final — whole remaining spec + carried <finding ids>
Phase <n>: scope — final — closure: carried <finding ids>
Phase <n>: repositories — <repo-a> [<branch>@<base7>]; <repo-b> [<branch>@<base7>]
Phase <n>: implemented — <repo-a> <base7>..<head7>; <repo-b> <base7>..<head7> — verified: <summary>
Phase <n>: accepted — <satisfied scope>
Phase <n>: accepted — <satisfied scope> — carry <finding ids>
Build: complete — <repo-a> <first-base7>..<head7>; <repo-b> <first-base7>..<head7>
```

Conditional entries:

```text
Phase <n>: concern — <verbatim implementer concern>
Phase <n>: review — blocking
F<n> | critical|important|minor | <self-contained finding>
Phase <n>: review — carried
F<n> | minor | <self-contained finding>
Phase <n>: re-review <R> — carried — <finding ids>
F<n> | minor | <new self-contained finding; new lines only>
Phase <n>: carried verdicts — <F<n> addressed|invalid|not_addressed; ...>
Phase <n>: final review repositories — <repo [first-base7..head7]>; ...
Phase <n>: review — final findings — coverage: <requirement/criterion ids or names>
F<n> | critical|important|minor | <self-contained finding>
Phase <n>: review — final clean — coverage: <requirement/criterion ids or names>
Phase <n>: fix round <R> repositories — <repo-a> [<branch>@<base7>]; ...
Phase <n>: fix round <R> concern — <verbatim fixer concern>
Phase <n>: fix round <R> — <repo-a> <base7>..<head7>; ... — verified: <summary>
Phase <n>: re-review <R> repositories — <repo [first-base7..head7]>; ...
Phase <n>: re-review <R> — open: <finding ids>
Phase <n>: re-review <R> — clean
Phase <n>: re-review <R> — open: <finding ids> — coverage: <final coverage>
Phase <n>: re-review <R> — clean — coverage: <final coverage>
Phase <n>: review: parked — F<n> — <authority, reason, and risk>
Phase <n>: acceptance pending — <acceptance ids or names> — <external dependency>
Phase <n>: acceptance verified — <acceptance ids or names> — <evidence>
Build: abandoned — <reason>
```

### Carried open set

Each finding has one complete ledger line under its first review/re-review anchor. Later entries use its stable id.

Compute open carried findings by adding ids in accepted `carry` suffixes, then removing only ids later marked `addressed`, `invalid`, or validly parked. `not_addressed` remains open. Completion rejects any carried id without one unique source line, reviewer verdict, or final owner.

## 4. Choose the current phase

Read the complete spec, ledger, and relevant code. Determine satisfied and remaining requirements from accepted evidence and current repository state. Start with all remaining requirements plus every open carried finding.

Take an intermediate boundary only when it materially reduces risk or cognitive load, permits an external transition to settle, or preserves the headroom needed for a reliable run. Every boundary pays another dispatch, review, fix loop, and full-suite run.

An intermediate landing must be independently safe to merge or deploy, preserve compatibility unless the spec changes it, contain no temporary half-migration, and provide observable acceptance evidence. Choose the largest coherent stable increment before the worthwhile boundary.

Split at valuable risk or cognition seams, not at technical layers or repository boundaries. One observable behavior spanning repositories is normally one phase; do not assume a particular repository layout. Absorb low-risk requirements that share implementation context, risk, or verification, and merge candidates that share tests, an invariant, or a transactional landing unless doing so would weaken landing safety or reliable-run headroom.

The phase is final when its frozen scope owns every remaining requirement and open carried finding and no known later stable landing is required. A one-phase build has final P1. If requirements are already satisfied but carried findings remain, choose a final closure phase.

For an intermediate phase, write and freeze only the current brief:

```md
# P<n> — <observable outcome>

## Requirements

<included spec requirements, preserving identifiers when present>

## Carried findings

<open ids, or omit this section when none>

## Landing

<independently deployable outcome and exact stop boundary>

## Verify

<spec acceptance criteria plus checks for carried findings>
```

If a carried finding lives in a repository otherwise outside the next phase, include that repository with a new phase base. A later phase may reopen an accepted phase's repository when its frozen scope requires it.

## 5. Dispatch and implementation

Before the first implementer dispatch, append `Phase <n>: repositories` with every phase repository's canonical path, branch, and current HEAD as its phase base. Before a successor implementer dispatch that adds a repository, append a new repositories entry with the complete expanded set and the new repository's phase base.

### Immutable manifests

Write one manifest per dispatch under `IMPLEMENT_DIR/dispatch`:

```text
P1-implement-D1.md
P1-review-D1.md
P1-fix-R1-D1.md
P1-rereview-R1-D1.md
```

Every manifest ends with `Manifest-complete: yes` and contains:

```text
# Dispatch
Manifest version: 1
Role: implementer|reviewer|fixer|re-reviewer
Mode: intermediate|final
Action: execute|evidence-recovery
Phase: P<n>
Generation: D<n>
Spec: <path>
Ledger: <path>
Scope: brief: <path>
# or: Scope: ledger: <path> | entry: <exact frozen scope entry>
...
Manifest-complete: yes
```

Required role fields:

| Role | Additional fields |
|---|---|
| Implementer | branch and phase base per repository; incoming carried ids grouped by ledger source anchor; evidence output path |
| Reviewer | reviewed ranges; implementation/acceptance evidence sources; temporary package path plus regeneration ranges; carried ids grouped by source anchor; first new finding id |
| Fixer | authorized branch and fix base per repository; supplied finding ids grouped by source anchor; reviewed package path/ranges; acceptance evidence sources; evidence output path |
| Re-reviewer | supplied ids grouped by source anchor; fixer/acceptance evidence sources; fix ranges; package path/regeneration ranges; first new id; new-finding admission rule |

For reviewer and re-reviewer manifests, set the first new finding id to one greater than the highest finding id already recorded anywhere in the ledger, or `F1` when the ledger has none.

Role contracts enforce general repository safety. Reference project-specific warnings through the spec, brief, or a ledger ruling. Manifests name finding ids and source anchors; record each new complete finding line in the ledger before dispatching its fixer or later owner.

Manifests are immutable. Reuse a complete manifest after a crash only when state and every referenced input are unchanged. A correction, repository expansion, changed input, or regenerated temporary package creates the next `D<n>`; never edit the prior file. Semantic changes append a ruling; routine package regeneration creates only a successor manifest.

Reject a manifest with an unknown version/role/mode/action, missing required field, unauthorized repository, or missing terminal marker.

### Agent-owned evidence

Implementer and fixer write structured evidence to the single path authorized by their manifest. The minimum format is:

```text
# Evidence
Dispatch: <manifest path>
Status: DONE|DONE_WITH_CONCERNS|BLOCKED

REPOSITORY:
Path: <canonical path>
Branch: <name>
Base: <sha>
Head: <sha>
Tests: <commands and results for this repository>
END_REPOSITORY

Integration: <commands and results, or n/a>
Acceptance: <checks and results>
Concern: <self-contained concern; repeat or omit>
Evidence-complete: yes
```

Repeat the repository block; keep integration and acceptance build-wide. `BLOCKED` also names the blocker and remaining work. The agent returns only:

```text
STATUS: <status>
EVIDENCE: <authorized path>
REPO_HEAD: <canonical path> | <sha>
```

Mechanically verify the evidence marker and schema; path equality; expected repository/head/branch/base/ancestry; clean trees; phase trailer on changed implementer heads; and exact review ranges. Append concerns and one compact implemented/fix ledger summary. Do not copy evidence into another file.

`Action: evidence-recovery` is implementer/fixer-only. It reruns or recollects evidence at immutable named heads without source changes and writes a successor evidence file.

For incomplete implementation, inspect committed work and resume the same implementer when safe. If another repository is required, add it with a phase base and successor manifest. If the approved spec cannot be satisfied, append an abandonment ruling and `Build: abandoned`, keep the implement directory as the terminal record, and stop.

## 6. Review, fix, and re-review

Generate one temporary `scripts/review-package` package for the active review manifest and preserve the repository ranges needed to regenerate it.

For every review or re-review supplied carried ids, append the complete `carried verdicts` summary before deciding its outcome; the latest verdict controls the open set. Anchor only findings first admitted by the current result under its `blocking`, `carried`, or `open` entry. Supplied findings retain their original source lines and are referenced by id.

Before the first `Action: execute` fixer dispatch for any round, append `Phase <n>: fix round <R> repositories` with each authorized repository's canonical path, branch, and current HEAD as its fix base. If a successor execute manifest adds a repository, append the complete expanded set; evidence recovery reuses the recorded bases.

A finding that proves an approved requirement is unsatisfied or the landing is unsafe is contract-blocking and cannot be carried or parked. Treat it as Important when the reviewer returned Minor.

### Intermediate phase

Dispatch the reviewer over exact phase ranges. It returns a verdict for every incoming carried id and a finding block.

- Clean, meaning no new findings and every carried id is `addressed` or `invalid`: accept the phase.
- Minor-only, including `not_addressed` carried ids: append `review — carried`, the new complete lines, and `accepted ... — carry <open ids>`; dispatch no fixer.
- Any Critical/Important: append `review — blocking` followed by the wave's new complete lines, including new Minor findings; `not_addressed` carried ids join the wave through their existing source anchors. Automatically authorize fix round 1.

The fixer receives all ids by ledger anchor, never copied lines. After a verified fix, dispatch one fresh re-reviewer over the fix ranges. It gives a verdict for every supplied id and may admit only fix-caused new findings.

- Any original/new Critical or Important remains blocking and enters the user gate.
- With no Critical/Important, every original `not_addressed` or new Minor is carried and the phase is accepted.
- If all supplied findings are addressed/invalid and no new finding exists, accept cleanly.

Only fix round 1 is automatic for an intermediate phase.

### Final phase

For a stable set of final heads, append `Phase <n>: final review repositories` with every cumulative first-build-base-to-current-HEAD range, then begin closure with one unrestricted whole-spec review at those heads using one cumulative package.

Use the manifest's evidence paths and ledger anchors as acceptance-evidence sources. The reviewer re-derives the complete requirement/criterion checklist from the spec. Record its returned coverage under `final findings` or `final clean`; use exact section/criterion names when the spec lacks ids.

A final `not_addressed` carried id joins the final fixer wave through its original source anchor.

Any final finding, including Minor or `not_addressed` carried work, automatically authorizes final fix round 1. The final fixer may change any participating repository required by the complete wave.

Before each final re-review, verify the latest heads, append `Phase <n>: re-review <R> repositories` with every cumulative first-build-base-to-current-HEAD range, regenerate the cumulative package, and re-derive the spec checklist. The re-reviewer gives every supplied verdict, reconfirms whole-spec coverage at those heads, and admits a new finding only when:

- the fix caused it; or
- it names the exact requirement, acceptance criterion, integration, compatibility, or safety coverage claim it invalidates.

Final round authority:

1. Initial final findings authorize round 1 for all severities.
2. A Minor-only result after re-review 1 authorizes ordinal round 2 once.
3. Any user-authorized round 2 consumes that allowance; no later round is automatic.
4. Critical/Important after re-review goes to the user gate.
5. If the user parks all eligible round-1 Critical/Important findings and only Minor findings remain, automatic ordinal round 2 is still available.
6. After ordinal round 2, the controller may park a genuine residual Minor only with reason, residual risk, and evidence that it affects no requirement, acceptance criterion, compatibility promise, or safety property. Otherwise promote it to Important.

Record controller parking as:

```text
Phase <n>: review: parked — F<n> — Ruling: parked by controller — <reason>; residual risk: <risk>; no requirement, acceptance, compatibility, or safety impact: <evidence>
```

## 7. User gate

Only the user may accept/defer Critical or Important risk, authorize a non-automatic round, or abandon for review reasons:

| Choice | Record and continue |
|---|---|
| Fix again | `Ruling: user authorized fix round <R> — <reason when given>` |
| Accept risk | `Phase <n>: review: parked — F<n> — Ruling: accepted by user — <reason and risk>` |
| Defer | `Phase <n>: review: parked — F<n> — Ruling: deferred by user — <reason and cost>` |
| Stop | append a ruling and `Build: abandoned — <reason>` |

Present the complete open wave. A user-authorized round handles every open severity. Contract-blocking findings cannot be parked; fix them, stop, or supersede the spec. The user may park other Critical/Important risk with a recorded reason and residual risk while preserving the accepted contract and safe landing.

## 8. Finish

Finish on coverage, not predicted phase count. Confirm:

- every requirement is covered by accepted scope and review evidence;
- the latest final review/re-review coverage entry covers the whole spec and is paired with cumulative repository ranges at every current HEAD;
- every carried id has one source, a verdict, and no open owner;
- every unparked finding is closed and every parked finding has the required authority/evidence;
- every tree is clean and every current range is reviewed; and
- no acceptance evidence remains pending.

If external acceptance evidence is pending, record the dependency and verification procedure and stop without completing. When evidence becomes available, append its verification and repeat the finish gate.

If a current repository HEAD is beyond the ranges paired with the latest final coverage entry, invalidate that final-review sequence. Record the new cumulative ranges, regenerate the package, and restart with an unrestricted whole-spec review at the current heads.

Append `Build: complete`, collect the result, then delete only this spec's implement directory. Report repository ranges, outcomes, tests, acceptance evidence, carried closures, parked findings, and rulings. Integration remains the user's next decision.

## 9. Resume

Read the spec and ledger, verify its first line names the spec, then verify every recorded commit, branch, head, range, tree, manifest, and required evidence file. If a recorded commit is reachable only from another branch, name that branch and ask before checking it out or continuing. Trust durable state over conversation memory. A terminal ledger never resumes automatically.

- Incomplete/missing manifest or unknown required field → do not dispatch; append a recovery ruling and create the next generation. Never repair it in place.
- Missing temporary package → regenerate from recorded ranges and create a successor manifest without a routine ruling.
- Missing, malformed, or schema-invalid evidence, including evidence without `Evidence-complete: yes` → treat it as lost. At unchanged verified heads, dispatch `evidence-recovery`; never substitute the ledger summary. If heads moved, use normal repository recovery.
- Scope exists without complete implementation → inspect status, log, and phase commits; verify finished work or resume implementation.
- Implemented line without review result → repeat the appropriate phase or whole-spec review.
- Complete `review — carried` without accepted line, with unchanged reviewed heads/scope → append accepted directly with anchored ids; do not rerun review.
- Blocking review without fix → run its authorized round 1. User authorization without a later fix → run that wave.
- Completed fix without re-review → run the matching intermediate or final re-review.
- Open Critical/Important after re-review → user gate. Final round-1 Minor-only → automatic ordinal round 2. Later Minor-only → controller parking rule or user authorization.
- Intermediate accepted with remaining work → choose again from actual heads. No requirements but open carried ids → final closure phase.
- Final clean/validly parked and all finish checks pass → complete.
- `Build: complete` with a leftover implement directory → collect the recorded result, delete only that spec's directory, and do not repeat completion work.
- `Build: abandoned` → report the recorded terminal state and keep the implement directory.

For a legacy ledger, exclude `review: minor deferred` entries from the carried set and derive finality from frozen scope, accepted requirements, and open carried ids without backfilling entries. If the ledger/run directory is lost, inspect code/history against the spec, repeat uncertain verification/review, then use verified heads as the recovery baseline; commit trailers alone do not establish acceptance.
