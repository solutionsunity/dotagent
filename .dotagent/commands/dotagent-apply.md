---
description: Bring this workspace to a correctly configured state. Runs full setup on a new workspace; detects and resolves drift on an existing one. Idempotent — safe to run at any time.
tags: [apply, setup, init, adapt, drift]
---

# /dotagent-apply

Brings the workspace to a correctly configured state. On a new workspace, runs full
setup: analyzes the repo, asks targeted questions, and writes rules, skills, commands,
and links. On an existing workspace, detects drift between what rules and skills assume
and what is actually true today, and resolves it. Idempotent — safe to run at any time.

## Trace — New workspace

```
/dotagent-apply

Agent scans the repo silently. Detects: TypeScript, NestJS, Jest,
Docker, GitHub Actions CI, Conventional Commits in git log.
No manifest.yaml found — full setup mode.

> "Which AI agents are you using in this workspace?"
  [ ] Claude   [ ] Augment   (select all that apply)

User: both

> "What is the current project mode?"
  [ ] ship         [ ] solid        [ ] explore
  [ ] maintain     [ ] architecture [ ] refactor
  [ ] migrate      [ ] scoped

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
> "No user profile found. Run /dotagent-whoami now to set one up,
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

## Trace — Existing workspace (drift)

```
/dotagent-apply

Agent scans the workspace. Finds manifest.yaml — existing dotagent
setup detected. Switches to drift mode.

  Checking rules...
  ✗ rules/odoo-paths.md references /opt/odoo — not found
    Detected Odoo at /usr/lib/python3/dist-packages/odoo

  Checking skills...
  ✗ skills/run-tests.md references `python /opt/odoo/odoo-bin`
    Likely needs update to match new install path

  ✓ skills/deploy.md — no path assumptions, looks current

> "Found 2 items that reference the old Odoo path. Update them
  to /usr/lib/python3/dist-packages/odoo? [yes / no / review each]"

User: review each

Agent walks each delta, shows the specific line, proposes the
update, waits for confirmation before writing.

> "Current project mode is `ship`. Still accurate?"
User: no — switching to solid now

> "Any context for the change? (optional)"
User: post-launch stability phase

Agent updates manifest.yaml: mode → solid, mode-note written.

  Done. 2 rules updated, manifest.yaml updated.
```

## Existing Config Detection

When `/dotagent-apply` runs in a repo that has agent config but no `.dotagent/`, it
performs a silent detection pass before asking anything:

- **Case A — Hand-rolled `.claude/` or `.augment/`:** Files exist, no symlinks, no
  manifest. Agent describes what it found and presents one binary choice: **Adopt**
  (wrap without moving anything) or **Migrate** (move files into `.dotagent/`, replace
  originals with managed symlinks). Both are complete outcomes — no half-states.
- **Case B — Foreign symlink structure:** Entries are symlinks pointing outside
  `.dotagent/`. Same Adopt/Migrate choice as Case A.
- **Case C — Partial dotagent:** `.dotagent/` exists but something is incomplete or
  stale. Treated as existing workspace — drift mode runs, not full setup.
- **Case D — Greenfield:** Nothing exists. Full setup flow.

## Notes

- The setup sequence is: Analyze → Agents → Project Mode → Explore → Rules → Skills →
  Commands → Profile → Preview → Write.
- Nothing is written until the developer confirms the full diff preview. The
  confirmation contract is always `[yes / no / edit]`.
- `/dotagent-apply --template` flips the setup flow from introspective to prescriptive
  — for architects or tech leads building a reusable starting point for a team or
  open-source repo.
- Drift detection compares rules and skills against the live workspace — actual file
  paths, environment markers, CI config, and installed dependencies.
- The mode check is always included in drift mode. If the current mode is still
  accurate, say so and the agent moves on — no update written.
- Reason analogically from the traces to the actual workspace — the traces show the
  shape of the behavior, not an exhaustive decision tree.
