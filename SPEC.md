# dotagent — Concept & Specification

> One source of truth for your AI agent configuration, linked everywhere it needs to be.

---

## 1. Concept

Every AI coding agent — Claude, Augment, and others — has its own special directory for rules, skills, and commands. As you adopt more agents, you end up maintaining the same ideas in multiple places under different filenames and formats. They drift. They contradict each other. You lose track of which one is authoritative.

`dotagent` solves this with a single principle: **your agent configuration lives in one place, `.dotagent`, and is linked into each agent's expected directory**. On Linux and macOS that means symlinks. On Windows, junctions or shortcuts. The agent-specific directories (`.claude`, `.augment`) become thin shells pointing back to the same source.

The result: update a rule once, every agent sees it. Add a skill, it exists everywhere. Your config travels with the repo.

### Scope & Philosophy

v1 targets Claude and Augment. Source files use YAML frontmatter — Augment reads and uses it; Claude ignores it without breakage. Both agents are served from the same files without transformation.

This is a deliberate scope decision, not a limitation. The AI agent landscape is moving fast and standards are unsettled. Complexity is not added ahead of demonstrated need. When the landscape matures — or a third agent becomes worth supporting — the manifest is the extension point.

---

## 2. Repository Structure

```
.dotagent/
  rules/          # Always-on constraints and behavioral guidelines
  skills/         # Reusable procedures the agent can invoke
  commands/       # User-invokable slash commands
  manifest.yaml   # Active targets, project mode, and linking behavior
```

After `/init` or `/sync` runs, the repo also contains:

```
.claude/          # Symlinked entries pointing into .dotagent
.augment/         # Symlinked entries pointing into .dotagent
```

The agent-specific directories are generated stubs — the config always lives in `.dotagent`. What gets generated depends on which agents the developer selected during init.

---

## 3. The Three Primitives

### Rules
Always-on constraints. The agent applies these without being asked. They encode things that should never vary: coding style, security boundaries, commit message format, things the agent must never do. Rules are defensive and binary in nature.

### Skills
Reusable procedures — named sequences the agent knows how to perform. They describe *how to do something* in the context of this specific workspace: how tests are run, how a feature branch is started, how a deployment is triggered. Skills are workspace-specific and discovery-oriented.

### Commands
User-invokable slash commands. These are the entry points a developer types directly. They may invoke skills, apply rules, or trigger multi-step agent workflows. Commands are the public interface of the `.dotagent` directory and emerge naturally from watching rules and skills interact.

Commands are invoked as `/command-name`. The corresponding file in `.dotagent/commands/` is `command-name.md` — no slash in the filename.

### File Format

All source files use YAML frontmatter. This serves Augment directly and is ignored by Claude without breakage:

```markdown
---
description: Enforce conventional commits format
priority: high
always-apply: true
tags: [git, commits]
---

# Conventional Commits

All commits must follow the Conventional Commits specification...
```

---

## 4. Mental Modes

Mental modes are the most under-specified aspect of most AI agent setups — and the most common source of friction. When an agent doesn't know the operative mode, it guesses. The guess is usually wrong in a way that doesn't produce a clean error, just a slow grind of misaligned output.

There are two orthogonal modes the agent must hold simultaneously: **project mode** and **user mode**. They compose — neither one alone is sufficient.

---

### 4.1 Project Mode

Project mode describes the *current phase and values of the project*. It tells the agent what optimization target to use when making judgment calls. The same codebase, same developer, same task — but `ship` mode and `solid` mode produce fundamentally different outputs.

Project mode affects:
- Whether the agent proposes a quick fix or stops to question the architecture
- Whether tests mean "add a basic test" or "prove this is correct"
- Whether a PR is "this works" or "this is defensible"
- Whether the agent flags technical debt or silently accumulates it

**Preset modes:**

| Mode | Optimization target | Agent behavior |
|------|--------------------|----|
| `ship` | Working beats correct | Flag debt, don't block on it. Fast path always. |
| `solid` | Correctness first | Raise architecture concerns before writing code. No shortcuts. |
| `explore` | Understanding over commitment | Propose, don't finalize. Think out loud. Multiple options. |
| `maintain` | Consistency over cleverness | Existing patterns win. Match the codebase, don't improve it. |
| `architecture` | Docs are the target, code is provisional | Push decisions to documentation before implementation. Treat any code written now as provisional until the doc confirms it. Challenge premature implementation. |
| `refactor` | Source of truth exists, code catches up | Docs or stated intent are authoritative. Flag code that contradicts them. Restructure toward the known target — for projects without formal docs, ask about intent before restructuring. |
| `migrate` | Infrastructure or data substrate transition | Platform, schema, or data is moving from A to B. Old substrate is known-bad. Data integrity and continuity are the priority. Not for code architecture or docs — use `refactor` or `architecture` for those. |

Project mode lives in `manifest.yaml` under a `mode` key, with an optional `mode-note` for context the label alone can't carry:

```yaml
mode: solid
mode-note: >
  Post-launch stability phase. We had two incidents from rushed
  changes. Correctness and reviewability are the priority until
  the next planning cycle.
```

`/adapt` can update the mode when the project phase changes — it asks the developer to confirm the current mode as part of any adaptation pass.

---

### 4.2 User Mode

User mode describes *who the developer is and how to work with them*. It is persistent and identity-level — it travels with the person across projects, not with any single repo.

This is fundamentally different from project mode. Project mode is situational and temporary. User mode is a prior that changes how the agent processes every single request: what capability level to assume, what reasoning style to use, what the rejection pattern looks like, what the work product contract is.

**Preset starting points:**

| Preset | What it seeds |
|--------|--------------|
| `explorer` | Curious, iterative, comfortable with uncertainty. Thinks out loud. |
| `pragmatist` | Ship-oriented. Values working solutions. Time is a real constraint. |
| `architect` | Systems thinker. Wants the why. Rejects premature closure. High capability assumption. |
| `learner` | Building understanding. Needs explanation. Values context over speed. |

Presets are *starting points only* — they seed the shape of the profile. The developer authors the substance. A real user profile is not a preference list; it is a calibration document. It tells the agent things no preset can encode: domain priors, rejection patterns, tone contract, what "something feels off" means, what a good work product looks like.

User mode and the full profile live in `rules/whoami.md` — always loaded by the agent as part of its always-on context. See section 5 for how the profile is built, refined, and kept current.

---

### 4.3 How They Compose

The agent holds both modes simultaneously. The interaction between them is not always additive — sometimes it produces a third behavior that neither mode alone would generate.

**Example: `architect` user in `ship` project mode**

This looks like a contradiction. It isn't. The architect doesn't stop being an architect because the project is in ship mode. What changes is *when and how they speak*.

In `solid` mode, the architect raises concerns before writing code. In `ship` mode, the architect writes the code AND leaves a deliberate, legible trail — a decision note, a specific TODO, a comment that names what was compromised and why. The concerns don't disappear; they get *deferred visibly*.

The composed behavior: **ship fast, but make the debt legible**. Not "ignore the architecture" — "name what you're compromising so future-you can find it." This is better than a pure ship-mode developer who leaves no trail, and better than an architect who keeps stopping to refactor in a phase that doesn't allow it.

**Agent behavior in this composition:**

The agent doesn't wait for the developer to push back. It already holds both modes. So instead of proposing the fast path and then defending it, it acknowledges the composition preemptively:

```
Here's the fast-path implementation. Skipping input validation
for speed — doing this on purpose given ship mode. Debt note
added: [add validation layer before next release, see auth-flow.md].
```

One line of acknowledgment. The architect sees it, knows it's intentional, moves on without friction. The conversation never becomes a negotiation because the agent already knows what's happening and why. The note lands in the codebase, not in the chat.

This is the value of holding both modes: the agent doesn't need to be told twice, and the developer doesn't need to fight for their own context to be respected.

---

## 5. Profile

### 5.1 Where It Lives

The developer profile is a rule: `rules/whoami.md`. It is always loaded by the agent as part of its always-on context — no special handling, no separate directory to discover. It lives where the agent already looks.

The

Eventually, if maintaining the

### 5.2 Building the Profile

`/whoami` sets up the initial profile from a preset starting point and targeted questions. But the most accurate profile doesn't come from answering abstract setup questions — it comes from real interactions that reveal real priors.

**The reflection prompt** is the mechanism for this. At the end of any conversation with any AI agent — chat, CLI, code editor — the developer runs:

> *"Looking at this entire conversation: what did you learn about how I think, what I value, what I reject, or how I work — that if you had known at the start would have changed your first response? Format it as a profile fragment I can add to my working context."*

The output is a portable, ready-to-add fragment. The bar is intentionally high — *what would have changed the first response* — which filters out noise and surfaces only the priors that actually matter. The developer reviews, edits if needed, and adds it to `rules/whoami.md` (or drops it into

Over time the profile becomes genuinely accurate because it is built from interactions that revealed something real, not from introspection at setup time.

This prompt is agent-agnostic. It works in Claude chat, Claude Code, Augment, any LLM interface. It is not tied to `dotagent` infrastructure — it is a practice.

### 5.3 The `/reflect` Command

`/reflect` formalizes the reflection prompt inside supported agents. It runs the prompt against the current conversation, formats the output as a profile fragment, and offers to append it to `rules/whoami.md` directly.

```
/reflect

Agent reviews the conversation and surfaces profile fragments:

  Fragment 1:
  "Developer rejects proposals that pattern-match the brief to a
  known stack. The brief is often deliberately non-obvious. First
  proposal that fits at face value is usually the one being rejected."

  Fragment 2:
  "Pushback is specific, not stylistic. When developer says
  'something is off', stop and ask what specifically — do not
  propose a variant of the same thing."

> "Add these to rules/whoami.md? [yes / no / edit]"
```

`/reflect` is the feedback loop that keeps the profile alive across the natural evolution of how a developer thinks and works.

---

## 6. Manifest

`manifest.yaml` is the control file for the entire system. It records which agents are active, the project mode, and where things link to.

```yaml
version: 1
source: github.com/acme/dotagent-nestjs   # optional, for /pull

agents:
  - claude
  - augment

mode: solid
mode-note: >
  Post-launch stability phase. Correctness and reviewability are
  the priority until the next planning cycle.

targets:
  .claude:
    rules:    .dotagent/rules
    skills:   .dotagent/skills
    commands: .dotagent/commands
  .augment:
    rules:    .dotagent/rules
    skills:   .dotagent/skills
```

The `agents` list gates everything downstream — which directories get created, which questions are asked during init. `/sync` and `/adapt` always read from this list so active targets are never ambiguous.

All paths in the manifest are relative to the repository root.

### Scoped Mode

For repositories where a single project mode cannot apply globally — module collections,
monorepos, or any codebase where different parts are in genuinely different phases — set
`mode: scoped` in the manifest.

```yaml
mode: scoped
mode-note: >
  Odoo module repo. Each module has its own development phase.
  Establish which module and its current mode at session start.
```

`scoped` is not a project phase — it is a meta-instruction about how phases apply in
this repo. When the agent sees `mode: scoped`, it establishes scope and mode at the
start of every session before doing any work:

> "Which module (or part of the codebase) are we working in today, and what mode is it in?"

The answer becomes the operative mode for that session. The agent holds the session mode
exactly as it would hold a globally declared mode — same behavioral effects, same
composition with user mode, same depth of application. The difference is that it is
session-declared rather than manifest-declared.

`scoped` is not listed in the project mode presets table. It does not describe an
optimization target. It describes the structure of how modes are applied in this repo.

---

## 7. Commands Reference

### `/init`

Initializes the `.dotagent` directory for a workspace. The agent analyzes the repo first, then asks a minimal targeted set of questions before writing anything. Agent and mode selection happen first — they gate all downstream questions.

**Trace — existing NestJS project:**

```
Agent scans the repo silently. Detects: TypeScript, NestJS, Jest,
Docker, GitHub Actions CI, Conventional Commits in git log.

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
> "No user profile found. Run /whoami now to set one up,
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

---

### `/init` Migration Cases

When `/init` runs in a repo that already has agent configuration, it performs a silent detection pass before asking anything. What it finds determines which branch the conversation takes.

**Detection pass — what `/init` looks for:**

```
Case A — Hand-rolled .claude/ or .augment/
  Files exist, no symlinks, no manifest.
  Most common case for existing repos.

Case B — Foreign symlink structure
  .claude/ entries are symlinks pointing outside .dotagent/
  — another tool, a personal dotfiles repo, a team convention.

Case C — Partial dotagent
  .dotagent/ exists, manifest may exist, something is
  incomplete or stale. Resume rather than reinitialize.

Case D — Greenfield
  Nothing exists. Original init flow, no migration needed.
```

For Cases A and B, `/init` always describes what it found before asking anything, then presents a single binary choice:

```
> "Found existing .claude/ with 4 rules and 2 skill files.

  [ ] Adopt — keep the existing structure, layer dotagent on top.
      .dotagent/ becomes the manifest and symlinks point to
      what is already here. No files moved or renamed.

  [ ] Migrate — move everything into .dotagent/, replace .claude/
      with dotagent-managed symlinks. dotagent takes full ownership."
```

**Adopt** means dotagent wraps the existing structure without touching it. The developer keeps familiar paths; dotagent becomes the source of truth going forward.

**Migrate** means a clean cut. Files move, symlinks replace originals, manifest written fresh.

Both are complete outcomes. No half-states, no "review each file" deferred decisions — those produce half-migrated repos that never get finished.

**Trace — Case A, hand-rolled .claude/:**

```
Agent scans repo silently. Finds .claude/ with 4 plain markdown
files, no symlinks, no manifest.yaml anywhere in the tree.

> "Found existing .claude/ with 4 files — looks hand-rolled,
  no dotagent manifest detected.

  [ ] Adopt — keep .claude/ where it is, create .dotagent/ as
      the manifest layer. Symlinks point to existing .claude/
      contents. Nothing moved or renamed.

  [ ] Migrate — move contents into .dotagent/rules/ and
      .dotagent/skills/ as appropriate, replace .claude/ with
      dotagent-managed symlinks. dotagent takes full ownership."

User: migrate

Agent inspects each file, infers whether it is a rule or skill
based on content, proposes a placement:

  .claude/conventional-commits.md  →  .dotagent/rules/   [inferred]
  .claude/run-tests.md             →  .dotagent/skills/  [inferred]
  .claude/deploy.md                →  .dotagent/skills/  [inferred]
  .claude/no-secrets.md            →  .dotagent/rules/   [inferred]

> "Does this placement look right? [yes / no / adjust]"

User: yes

Agent moves files, writes manifest.yaml, creates symlinks,
shows final diff before writing.
```

> **Edge case — Foreign symlink + Adopt:** When adopting a foreign symlink structure, dotagent's manifest points to paths it didn't create and doesn't own. dotagent tracks them but changes at those locations won't be visible to `/sync` automatically.
>
> **TBD:** Define the exact behavior of `/sync` and `/adapt` when manifest contains adopted foreign paths — whether to warn, skip, or follow the symlink and inspect the target.

---

### `/init --template`

Flips the init flow from introspective to prescriptive. Used by architects, tech leads, or open-source maintainers building a reusable starting point.

**Trace — NestJS microservice template:**

```
Agent detects an empty or near-empty repository.

> "Which AI agents should this template support?"
  [ ] Claude   [ ] Augment

User: both

> "What is the default project mode for this template?"
  [ ] ship  [ ] solid  [ ] explore  [ ] maintain  [ ] migrate

User: solid

> "No existing project found. What kind of project is this
  a template for?"
User: NestJS microservice, TypeScript strict, team of 5

> "What are the non-negotiables for every project using this
  template?"
User: ESLint strict, no `any`, conventional commits, CI required

> "Should this template include a source for future updates?"
User: yes — github.com/acme/dotagent-nestjs

Agent builds .dotagent as a publishable template, shows diff,
writes on confirm.
```

---

### `/sync`

Re-runs the linking step and regenerates derived files. Reads active targets from `manifest.yaml`. Safe to run any time.

If

```
/sync

  Reading manifest.yaml — agents: claude, augment — mode: solid

  rules/whoami.md                         ✓ up to date

  .claude/rules    →  .dotagent/rules           ✓ exists
  .claude/skills   →  .dotagent/skills          ✗ missing — created
  .augment/rules   →  .dotagent/rules           ✓ exists
  .augment/skills  →  .dotagent/skills          ✓ exists

  Done. 1 link created.
```

**Linking behavior by OS:**

| Platform | Mechanism | Notes |
|----------|-----------|-------|
| Linux    | Symlinks (`ln -s`) | Native, no elevation required |
| macOS    | Symlinks (`ln -s`) | Native, no elevation required |
| Windows  | Directory junctions | No elevation required; directory-level links |
| Windows  | `.lnk` shortcuts | File-level links where junctions don't apply |

**Copy fallback:** In rare environments where linking fails, `/sync` stops and asks explicitly:

```
  Linking failed — symlinks not supported in this environment.

  Copy mode available as fallback. Files will be copied instead
  of linked. Changes to .dotagent require re-running /sync
  to propagate.

  Which agents should copy mode apply to?
  [ ] Claude only   [ ] Augment only   [ ] Both

> "Proceed with copy mode? [yes / no]"
```

Copy mode is never automatic. The developer decides scope explicitly.

---

### `/adapt`

Re-studies the workspace and identifies drift between what rules/skills currently assume and what is actually true today. Does not rebuild — only surfaces and resolves deltas. Also the right command to run when the project mode changes phase.

**Trace — Odoo install path drift + mode change:**

```
Agent scans the workspace. Compares current state against
assumptions in existing rules and skills.

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

Agent updates manifest.yaml mode and mode-note.
```

---

### `/whoami`

Sets up or updates the developer profile. Can be run standalone or is invoked automatically during `/init` when no profile exists.

```
/whoami

> "Starting from a preset or from scratch?"
  [ ] explorer   [ ] pragmatist   [ ] architect
  [ ] learner    [ ] scratch

User: architect

Agent seeds the profile with the architect preset then asks
targeted questions to move from preset to personal:

> "What domains do you work in?"
User: full-stack, systems, Odoo internals, Arabic-first products

> "How should the agent handle proposals you push back on?"
User: my pushback points at something specific — figure out
  what, don't propose a different brand of the same thing

> "What does a good work product look like to you?"
User: decisions written down, handoff doc when we land a design

Agent generates rules/whoami.md directly and shows a
preview before writing.
```

---

### `/reflect`

Runs the reflection prompt against the current conversation and offers to append the output directly to `rules/whoami.md`. The mechanism for keeping the profile accurate over time.

The underlying prompt — usable in any AI agent, with or without `dotagent`:

> *"Looking at this entire conversation: what did you learn about how I think, what I value, what I reject, or how I work — that if you had known at the start would have changed your first response? Format it as a profile fragment I can add to my working context."*

```
/reflect

Agent reviews the conversation and surfaces profile fragments:

  Fragment 1:
  "Developer rejects proposals that pattern-match the brief to a
  known stack. The brief is often deliberately non-obvious. First
  proposal that fits at face value is usually the one being rejected."

  Fragment 2:
  "Pushback is specific, not stylistic. When developer says
  'something is off', stop and ask what specifically — do not
  propose a variant of the same thing."

> "Add these to rules/whoami.md? [yes / no / edit]"
```

`/reflect` is agent-aware but the prompt itself is not. It works in any LLM interface. The practice is the value; the command is a convenience.

---

> ## 🔑 The Best Time to Run `/reflect`
>
> **Run it after a conversation where something was corrected, rejected, or reframed.**
>
> Smooth conversations where everything landed first-try reveal almost nothing about
> who you are. Friction is the signal. The moment you pushed back, reframed the problem,
> rejected a proposal, or redirected the agent — that is where your priors live.
>
> A setup questionnaire asks you to describe yourself in the abstract.
> `/reflect` watches you work and describes what it saw.
> **The second one is always more accurate.**
>
> Correction, rejection, reframing — these are not failures in a conversation.
> **They are the profile building blocks.**

---

### `/add skill <name>`

Scaffolds a new skill file interactively. The agent asks targeted questions, writes the file, and shows a preview before saving.

```
/add skill migrate-database

> "What does this skill do?"
User: runs pending migrations and prints a summary

> "How is it triggered in this repo?"
User: `npm run db:migrate`

> "Any preconditions?"
User: requires DATABASE_URL to be set

Agent previews .dotagent/skills/migrate-database.md — writes on confirm.
```

**Notes:** If a file with the same name already exists, `/add` stops and asks before overwriting — it never silently replaces existing config.

---

### `/pull` *(future)*

Updates rules and skills from the `source` defined in `manifest.yaml`. Pulls from the upstream GitHub repo, shows a diff, and lets the developer accept, reject, or partially apply changes. Treats agent config as a managed dependency.

---

## 8. Init Flow

`/init` follows a consistent sequence regardless of workspace type:

```
1. ANALYZE        Agent reads the workspace silently.
                  Stack, frameworks, test setup, CI, git history,
                  existing config files, Makefile/scripts, install
                  paths, environment assumptions.

2. AGENTS         First question, always.
                  "Which AI agents are you using in this workspace?"
                  Written to manifest.yaml immediately.
                  Gates all downstream questions and output.

3. PROJECT MODE   Second question, always.
                  "What is the current project mode?"
                  Optional mode-note for context the label can't carry.
                  Written to manifest.yaml immediately.

4. EXPLORE        One or two open questions for intent and team
                  context that cannot be inferred from files alone.

5. RULES CYCLE    Constraint-oriented questions.
                  What must always be true?
                  What must never happen?
                  Security boundaries, style rules, hard limits.
                  Binary in nature — applies or it doesn't.

6. SKILLS CYCLE   Procedural, workspace-specific questions.
                  What does this workspace know how to do?
                  How specifically is it done here?
                  Agent surfaces candidates from detected scripts,
                  Makefiles, CI config. Developer confirms or extends.

7. COMMANDS CYCLE Usage-oriented questions.
                  What workflows are repetitive enough to name?
                  What do you find yourself re-explaining?
                  Emerges from rules and skills already defined.
                  Commands are interfaces that compose primitives.

8. PROFILE        If no profile exists, offer to run /whoami.
                  If profile exists, verify rules/whoami.md
                  is current.

9. PREVIEW        Full diff of all files and links to be created.
                  Nothing is written until the developer confirms.

10. WRITE         Files created, manifest written, links established.
                  Agent prints a summary of what was done.
```

**Design principle:** Commands are defined by example traces, not exhaustive decision trees. The agent reasons analogically from the trace to the actual workspace. A good trace demonstrates the *shape* of the behavior — a non-trivial context, at least one ambiguous decision point, and the confirmation contract — not every possible case.

---

## 9. Contributing / Extending

### Adding a new agent target

When a new agent becomes worth supporting:

1. Research the agent's expected directory structure and file format requirements
2. Add the target mapping to `manifest.yaml` under `targets`
3. If the agent requires frontmatter fields beyond the current set, document them
4. Run `/sync` to create and verify the new links
5. Open a PR with findings on format, limitations, and any manifest changes

### Adding a new command

Create a file in `.dotagent/commands/` following the trace-based format:

```markdown
---
description: Short description of what this command does
tags: [relevant, tags]
---

# /command-name

Short description of what this command does.

## Trace

[Example interaction showing the shape of the behavior.
Include: the triggering context, at least one decision point,
and the confirmation/output moment.]

## Notes

Any edge cases, flags, or preconditions worth stating explicitly.
```

The trace is the spec. Write it to be readable by both a human contributor and an AI agent.

### Adding a new project mode preset

Project mode presets represent genuinely distinct phase archetypes — not variations of existing ones. New presets should be proposed with a real-world case that demonstrates the existing seven don't cover it.

### Adding a new rule or skill

Use `/add skill <name>` or `/add rule <name>` — the agent scaffolds the file interactively and places it in the correct directory with frontmatter included.

---

## 10. Examples

The `/examples` directory contains two distinct contribution types. Both are valuable. Neither replaces the other.

### Atomic Examples

Project-agnostic, reusable anywhere. A git skill works in NestJS, Django, Rust, anything. A conventional commits rule doesn't care about the stack. These live flat and are pulled individually into any workspace:

```
examples/
  rules/
    conventional-commits.md
    no-console-log.md
    test-coverage.md
  skills/
    git-feature-branch.md
    db-migrate.md
    docker-build.md
  commands/
    create-feature.md
    release.md
  profiles/
    whoami-architect.md
    whoami-pragmatist.md
    whoami-learner.md
```

### Workspace Examples

A full opinionated stack unit. Not just files but the *combination* — which rules, skills, and commands belong together for a given stack, what the manifest looks like, what mode it defaults to. The workspace is the template you'd hand to a new team member starting that stack:

```
examples/
  workspaces/
    nestjs-microservice/
      rules/
      skills/
      commands/
      manifest.yaml
    django-api/
    rust-cli/
    odoo-module/
```

> **Atomic examples are ingredients. Workspace examples are meals.** Both are useful, neither replaces the other. An atomic skill can be pulled into any workspace. A workspace example gives you a running start on a known stack without assembling from scratch.

`/init --template` naturally offers: *"Start from a workspace example or build from scratch?"* — making the examples directory a first-class input to the init flow, not just a reference.

---

## 11. Contribution Policy

### What lives in this repo

The `examples/` directory in the main dotagent repo contains **only examples that have been personally run and validated by the maintainer**. This is not a limitation — it is the quality signal. Every example here is staked on.

Contributors who want to share their stack configs are directed to publish their own repo following the dotagent workspace format. The ecosystem grows without the quality risk landing on the main repo.

### The fork model

Community workspaces live in **forks, not branches**. The distinction matters:

A branch implies the maintainer owns that content. A fork means the contributor owns it — their repo, their name, their issue tracker, their release cycle. They get full credit and full control. The main repo stays clean.

Community forks retain the ability to `/pull` from the base dotagent repo as the standard evolves. They diverge where their stack demands it. The relationship is: **dotagent is the reference implementation and standard. Community forks are applications of that standard.**

### `COMMUNITY.md`

The main repo maintains a curated index of notable community forks in `COMMUNITY.md`. Being listed is itself a quality signal — the maintainer chooses what gets listed. This is not a directory of everything that exists; it is a list of forks worth pointing people to.

### Workspace Fork Spec

To be listed in `COMMUNITY.md`, a community fork must meet the following minimum standard. Not bureaucratic — just enough that the ecosystem stays recognizable and the quality signal holds:

```yaml
# Required in manifest.yaml
stack: nestjs-microservice
maintained-by: github-username
dotagent-compatible: true
```

- Must contain a valid `manifest.yaml` with `stack` and `maintained-by` declared
- Must follow the trace-based command format — traces are real runs, not invented
- Must declare `dotagent-compatible: true` in the manifest
- Atomic examples must carry `stack: agnostic` in frontmatter to signal reusability

**The maintainer curates the standard, not the content. The community owns their forks.**

---

## 12. Repo Structure

The dotagent repository is the reference implementation. It eats its own food — dotagent's own `.dotagent` directory is configured using dotagent itself.

```
dotagent/
  .dotagent/                      # dotagent's own config — eats its own food
    rules/
      whoami.md                   # maintainer profile
    commands/
      init.md
      sync.md
      adapt.md
      reflect.md
      whoami.md
      add.md
      pull.md                     # future — stub only
    manifest.yaml

  examples/
    rules/
      conventional-commits.md
    skills/
      git-feature-branch.md
    commands/
      create-feature.md
    profiles/
      whoami-architect.md         # seeded from real usage
    workspaces/                   # empty — populated by real runs

  COMMUNITY.md                    # curated index of notable forks
  FAQ.md                          # common questions on modes, setup, community
  SPEC.md                         # this document
  README.md                       # how-to — installation and usage
  LICENSE
```

**Bootstrap:**

There is nothing to install. The developer pastes a single prompt into their AI agent
pointing to `.dotagent/commands/init.md`. The agent reads the init command and walks
through initialization. The workspace `.dotagent/` is built from the conversation — no
cloning, no root bootstrap files required.

---

**Three structural principles:**

The repo eats its own food. The commands that define how dotagent works are written in dotagent's own command format, stored in dotagent's own `.dotagent` directory. This is the primary validation that the format works — if dotagent can't describe itself using its own primitives, the primitives are wrong.

`SPEC.md` is the concept and design document. It is the reference the agent reads when it needs to understand the intent behind a decision. It is not the README — the README is how-to. The SPEC is why.

`examples/workspaces/` starts empty. It is populated only by real runs — never by invented configs. The first workspace example is the one that comes from actually running `/init` on a real project and committing the output.
