# Spec self-review checklist

Run after writing, before the user review gate. Read the spec with fresh
eyes — as the build window will: only this document and the workspace, no
conversation memory. Fix inline and move on — no second review pass.

1. **Conversation-only check:** walk the requirements one by one and ask: is
   the exact value or behavior stated in the document, or do I know it only
   from the conversation? Anything you know that the document doesn't say —
   write it in.
2. **Internal consistency:** do any sections contradict each other? Does the
   design match the requirements?
3. **Traceability:** every requirement uses a unique, stable
   `### R<n> — <name>` heading. Every acceptance criterion uses the
   `- A<n> (R...):` form, names the requirement or requirements it verifies,
   and collectively the criteria cover every requirement.
4. **Scope check:** does the spec describe one coherent goal? Do not split it
   merely because implementation may need several phases. Use separate specs
   only for goals that can be owned, approved and evolved independently.
5. **Ambiguity check:** any "TBD", "TODO", or requirement that could be read
   two ways? Resolve it per the gap rules in SKILL.md.
6. **Context check:** does section 3 capture the non-obvious current facts,
   affected systems or repositories, and constraints a fresh context needs,
   without turning into an exhaustive file inventory?
7. **Contract and transition check:** are external interfaces, compatibility
   invariants, migrations, rollout constraints and failure behavior settled
   or explicitly out of scope?
8. **Acceptance quality:** do the criteria verify observable end-state
   behavior rather than implementation steps? Are exact commands accompanied
   by the repository or working directory needed to run them, and are
   integrated or cross-repository checks included where the behavior requires
   them?
9. **No task or phase plan:** the spec contains no execution plan, task
   breakdown or predicted phase structure. Planning happens at build time on
   the live code.
10. **Metadata check:** there is no `Baseline:`, status or build-lifecycle
   line. `Supersedes:` appears only when this spec replaces an earlier one.
   User approval, not metadata or a commit, freezes the file.
