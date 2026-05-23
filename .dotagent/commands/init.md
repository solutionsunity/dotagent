---
description: Initialize the .dotagent directory for a workspace. Analyzes the repo silently, asks targeted questions, writes files and symlinks on confirmation.
tags: [init, setup, onboarding]
---

# /init

Initialize the `.dotagent` directory for a workspace. The agent reads the repo silently
first, then asks a minimal targeted set of questions before writing anything. Agent and
mode selection happen first — they gate all downstream questions and output.

## Trace

```
Agent scans the repo silently. Detects: TypeScript, NestJS, Jest,
Docker, GitHub Actions CI, Conventional Commits in git log.

> "Which AI agents are you using in this workspace?"
  [ ] Claude   [ ] Augment   (select all that apply)

User: both

> "What is the current project mode?"
  [ ] ship     [ ] solid     [ ] explore
  [ ] maintain [ ] migrate

User: solid

> "Any context behind that choice? (optional)"
User: post-launch, we had two incidents from rushed changes

--- Explore ---
> "I see Jest configured with no coverage threshold — should I add
  a coverage rule? (e.g. enforce 80% minimum)"
User: yes, 80%

> "Your git history follows Conventional Commits consistently.
  Lock that in as a required rule?"
User: yes

> "I found a Makefile with `make test` and `make deploy`. Should I
  build skills around those?"
User: yes, and also add a db:seed skill — here's what it does: ...

--- Rules ---
> "Any security boundaries agents must never cross?"
User: never commit .env files, never expose API keys in logs

> "Anything about code style not captured by the linter?"
User: no

--- Skills ---
> "Any other repeated workflows worth naming as a skill?"
User: no

--- Commands ---
> "Any workflows you find yourself re-explaining to the agent?"
User: yes — /create-feature, branch naming and PR template

Agent scaffolds /create-feature and asks two targeted questions.

--- Profile ---
> "No user profile found. Run /profile now to set one up,
  or skip and add it later? [setup / skip]"

User: skip

Agent shows a full diff preview:

  .dotagent/rules/conventional-commits.md       [new]
  .dotagent/rules/test-coverage-80.md           [new]
  .dotagent/rules/no-secret-exposure.md         [new]
  .dotagent/skills/run-tests.md                 [new]
  .dotagent/skills/deploy.md                    [new]
  .dotagent/skills/db-seed.md                   [new]
  .dotagent/commands/create-feature.md          [new]
  manifest.yaml                                 [new]

  Links to create:
    .claude/rules    →  .dotagent/rules
    .claude/skills   →  .dotagent/skills
    .claude/commands →  .dotagent/commands
    .augment/rules   →  .dotagent/rules
    .augment/skills  →  .dotagent/skills

> "Write these files? [yes / no / edit]"
```

## Migration Cases

When `/init` runs in a repo with existing agent config, it performs a silent detection
pass before asking anything:

- **Case A — Hand-rolled `.claude/` or `.augment/`:** Files exist, no symlinks, no
  manifest. Most common existing-repo case.
- **Case B — Foreign symlink structure:** Entries are symlinks pointing outside
  `.dotagent/` — another tool, a dotfiles repo, a team convention.
- **Case C — Partial dotagent:** `.dotagent/` exists but something is incomplete or
  stale. Resume rather than reinitialize.
- **Case D — Greenfield:** Nothing exists. Main flow, no migration needed.

For Cases A and B, the agent describes what it found, then presents one binary choice:
**Adopt** (wrap existing structure without touching it) or **Migrate** (move files into
`.dotagent/`, replace originals with managed symlinks). Both are complete outcomes — no
half-states, no deferred per-file decisions.

## Notes

- `/init --template` flips the flow from introspective to prescriptive — for architects
  or tech leads building a reusable starting point for a team or open-source repo.
- The init sequence is: Analyze → Agents → Project Mode → Explore → Rules → Skills →
  Commands → Profile → Preview → Write.
- Nothing is written until the developer confirms the full diff preview.
- The confirmation contract is always `[yes / no / edit]`.
- Reason analogically from the trace to the actual workspace — the trace is the shape
  of the behavior, not an exhaustive decision tree.
