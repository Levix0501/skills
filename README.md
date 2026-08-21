# skills

A doc-based build pipeline for coding agents: from vague idea to reviewed, verified build, with nothing load-bearing left in the conversation.

- **Interview before code.** `/kick-off` pins down requirements, edge cases and acceptance one question at a time — facts are looked up in the code, decisions stay with the user. Oversized ideas decompose into a `docs/roadmap/` queue, built one sub-project at a time.
- **Frozen specs.** Approval makes a spec read-only for every process. Git preserves it when available; otherwise the workflow warns that the workspace document is local-only. A requirements change is a new spec superseding the old one.
- **Fresh eyes at every step.** Fresh subagents implement each phase, review every landing, fix each finding wave and re-review every fix — no one referees their own work. Non-blocking Minor findings are carried into the next phase instead of disappearing from the build.
- **Coarse, durable checkpoints.** Requirements live in the spec, accepted phase landings and rulings in an append-only `progress.md`, and finished work in git. Immutable dispatch manifests and agent-owned evidence keep recovery state outside the conversation.
- **Verified, not trusted.** Intermediate landings remain independently shippable; the final landing receives a whole-spec final review at the current heads before completion.

## Install

```sh
npx skills add Levix0501/skills
```

Pick skills interactively, or non-interactively: `npx skills add Levix0501/skills -s implement -g -y`.

Manual alternative (Claude Code): copy a skill directory into `~/.claude/skills/` (user-level) or `<repo>/.claude/skills/` (project-level). Skills register at session start — open a new session after installing.

## Skills

| Skill | What it does |
|-------|--------------|
| [`kick-off`](skills/kick-off/) | A relentless interview that pins down what the user actually wants before anything gets built. |
| [`create-spec`](skills/create-spec/) | Writes the settled requirements into a spec a fresh window can build from alone. |
| [`implement`](skills/implement/) | Turns an approved spec into reviewed, independently shippable code through adaptive stable phases. |

## Pipeline

```
/kick-off <idea>     # explore → interview → convergence summary → spec or inline
/create-spec         # spec written, self-reviewed and user-approved — committed when Git-backed, always frozen
/implement docs/specs/<file>.md  # choose the next stable phase → implement → review and fix → repeat or finish
```

While an implementation is active, `/implement` keeps its recoverable parts outside the conversation: requirements in the approved, read-only spec; accepted phase state, carried Minor findings and rulings in `progress.md`; an immutable brief only for the current stable increment when one is needed; compact dispatch manifests and evidence in the run directory; and landed work in the affected Git repositories. The implement directory lives beside the spec at `<spec-dir>/.implement/<spec-name>/`. Start it in the same window as the spec or a fresh one. Kill the window mid-build — compact, crash, new machine session — then in any window:

```
/implement docs/specs/2026-08-11-foo.md   # or bare /implement
```

The build resumes from the first phase state without a terminal checkpoint. Git says what actually landed; missing manifests, temporary review packages or evidence are regenerated or recollected from durable ranges. After every spec requirement and carried finding is closed and a whole-spec final review covers the current repository heads, `/implement` removes the implement directory, leaving the spec and repository histories as the durable record.

For doc-driven work outside this flow, `/implement` works standalone.

## License

MIT.
