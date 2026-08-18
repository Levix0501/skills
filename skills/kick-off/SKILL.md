---
name: kick-off
description: A relentless interview that pins down what the user actually wants before anything gets built.
disable-model-invocation: true
---

Explore before you ask. Build the fact base first: a *fact* the codebase can answer is looked up — before the interview or mid-interview, whenever one surfaces — and never asked. The *decisions* belong to the user: put each one to the user and wait for an answer. When the user's answer conflicts with what the code actually does, show the evidence and ask the user to re-confirm.

## The queue

`docs/roadmap/` is the backlog of sub-projects earlier kick-offs decomposed but did not build: one file per pending sub-project, `NN-<slug>.md`, ordered by the two-digit prefix. The file's existence IS the state — created when a decomposition lands, deleted in the same commit that gives the sub-project its next durable carrier. No status fields anywhere; an empty directory is a finished roadmap.

Start every kick-off by checking it. If `docs/roadmap/*.md` exists, ask the user first (AskUserQuestion): continue with the lowest-numbered entry — name it — or start something new. When the user picks an entry, read it and use its Scope / Depends on / Context as the seed facts for the interview; it is a starting point, not a contract — the interview may reshape it.

## Assess scope

Before asking detailed questions, assess scope: if the request describes multiple independent subsystems (e.g., "build a platform with chat, file storage, billing, and analytics"), flag this immediately. Don't spend questions refining details of a project that needs to be decomposed first.

If the project is too large for a single spec, help the user decompose it: what are the independent pieces, how do they relate, what order should they be built? Once the user confirms the split, write it into the queue — one file per sub-project, numbered in build order (`docs/roadmap/01-auth-core.md`, `02-payment-gateway.md`, …). Each file is the seed for that sub-project's future interview, not a spec — a few lines:

```markdown
# <Name>

Scope: one or two sentences — what it covers, what it explicitly leaves out.
Depends on: 01 (or: none)
Context: decomposed YYYY-MM-DD from "<the original request>"; the overall goal in one line.
```

Commit the whole queue in one commit, then take the first entry and continue through the flow below. Each sub-project gets its own interview → build cycle.

The rule is recursive: when a queue entry the user picked turns out to be too large mid-interview, split it into `NN-1-<slug>.md`, `NN-2-<slug>.md`, … — inheriting the number so the order holds without renumbering — and delete the original in the same commit.

An entry leaves the queue only when its next durable carrier exists; abandoning one is deleting its file with the reason in the commit message. Never mark, move, or annotate an entry as done — deletion is the only exit.

## The interview

For appropriately-scoped projects, interview the user relentlessly about every aspect of this plan until there is a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer with one line of reasoning and what it costs.

Ask the questions one at a time, waiting for feedback on each question before continuing. Asking multiple questions at once is bewildering.

## The convergence summary

When the tree is walked, close the interview with a convergence summary:

- the problem, the goal, and the non-goals
- every confirmed requirement, with its exact value
- edge cases, failure behavior, compatibility constraints
- acceptance criteria
- assumptions and rejected alternatives
- the line "Open questions affecting implementation: none" — if you can't write it truthfully, the interview isn't done; ask what's missing

Present the summary and wait. The user's confirmation of it is the shared understanding — do not enact the plan before it.

## The next step

After the user confirms, ask which way to build (AskUserQuestion), two options:

| Choice | Then |
|---|---|
| `create-spec` | write the summary into a spec — invoke that skill and continue |
| Direct execution | build it immediately in this window, with the summary as the requirements record |

Judge which to recommend with the signals in [references/complexity-rubric.md](references/complexity-rubric.md) — any hard signal means spec. Put the recommended option first, with its 1–3 main reasons in the option description. The choice belongs to the user; wait for it.

If this kick-off started from a queue entry, the entry's exit rides the choice: `create-spec` retires it during the approval handoff — in the same commit when both files share a Git repository, otherwise only after asking before deletion. For direct execution, retire the entry through its owning repository before the build starts; the convergence summary is now the requirements record.

If mid-build the work sprouts a hard signal the rubric missed, stop expanding and recommend switching to a spec: the summary plus the work so far feed `create-spec` directly.
