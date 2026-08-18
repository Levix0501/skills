# Spec or inline: the complexity rubric

The question at the end of a kick-off interview: write a spec
(`create-spec` → the selected implementation workflow, a resumable reviewed build) or execute
inline in this window (fast, no artifacts, no resume). Judge by signals, not
by size — file counts are hints at most.

## Hard signals — any ONE present → recommend spec

- Data migration, or any operation that is hard to reverse
- Security, permissions, or billing surface
- Public interface or compatibility risk (API, CLI flags, file formats,
  other consumers)
- Spans multiple modules or an important seam between systems
- Expected to span sessions, or someone else must review the requirements
- The decision + acceptance matrix is large enough that losing one entry
  changes the outcome
- The work warrants the full review → fix → re-review discipline

## Inline conditions — essentially ALL must hold → inline is defensible

- The change is local and rollback is trivial (`git revert` and done)
- No migration, no contract or compatibility risk
- Finishable in the current session
- The convergence summary alone is an adequate short-term requirements
  record — nobody will need it next week

## Presenting the recommendation

Name the recommendation and its 1–3 decisive signals — not the whole
checklist. The user decides; both paths are legitimate.

## Misjudged inline

The rubric misfires in one direction that matters: work that looked local
starts sprouting hard signals mid-build. The rule: stop expanding
immediately and recommend converting to a spec — the interview summary plus
the work done so far feed `create-spec` directly. Pushing on "because we
already started" is how unreviewed migrations happen.
