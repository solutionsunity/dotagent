# dotagent

One source of truth for your AI agent configuration, linked everywhere it needs to be.

---

## Why dotagent

**If you use one AI agent** — dotagent gives you a principled structure for configuring it properly. A `whoami.md` that tells the agent who is operating it. Mental modes that tell it what phase the project is in. Trace-based commands that define how you work. Most `.claude` directories are a handful of half-thought rules. dotagent is the answer to how to set this up right.

**If you use multiple agents** — dotagent adds a symlink layer so your configuration lives in one place and is linked into each agent's expected directory. Update a rule once, every agent sees it. Add a skill, it exists everywhere. No drift, no duplication, no deciding which one is authoritative.

Both cases. One tool. The structure you build for a single agent just works everywhere when you add more.

## How It Works

Your agent configuration lives in one place — `.dotagent` — and is linked into each agent's expected directory.

```
.dotagent/
  rules/        # always-on constraints
  skills/       # reusable workspace procedures
  commands/     # slash commands you invoke directly
  manifest.yaml # active agents, project mode, linking config
```

`.claude`, `.augment` symlink into `.dotagent`. One source, many targets.

Only `rules/`, `skills/`, and `commands/` are symlinked — not the whole directory. Any agent-specific files you already have (or need to add) sit in `.claude/` or `.augment/` normally alongside the links.

---

## Quickstart

Open your AI agent in your project directory and paste this prompt:

```
Read https://raw.githubusercontent.com/solutionsunity/dotagent/main/SPEC.md
then run /init for this workspace.
```

The agent reads the spec, understands the format, and walks you through initialization.
Your `.dotagent/` is built from the conversation — nothing to clone, nothing to install.
The prompt is the bootstrap.

---

## What `/init` Does

1. Detects your stack, frameworks, CI, existing config
2. Asks which AI agents you use — gates everything downstream
3. Asks your current project mode (`ship`, `solid`, `explore`, `maintain`, `migrate`)
4. Walks through rules, skills, and commands — surfacing candidates, asking targeted questions
5. Offers to set up your developer profile (`whoami.md`)
6. Shows a full diff preview
7. Writes only on confirmation

**Existing `.claude` or `.augment`?** `/init` detects it and offers two choices: adopt (wrap without moving anything) or migrate (full ownership transfer). No half-states.

---

## Commands

| Command | What it does |
|---------|-------------|
| `/init` | Initialize `.dotagent` for a workspace |
| `/init --template` | Build a shareable stack template |
| `/sync` | Re-run linking, regenerate derived files |
| `/adapt` | Detect and resolve workspace drift |
| `/whoami` | Set up or update your developer profile |
| `/reflect` | Extract profile fragments from the current conversation |
| `/add skill <name>` | Scaffold a new skill interactively |
| `/add rule <name>` | Scaffold a new rule interactively |
| `/pull` | Pull updates from upstream source *(future)* |

---

## Mental Modes

dotagent introduces two modes the agent holds simultaneously:

**Project mode** — the current phase and optimization target of the project. Set in `manifest.yaml`, updated via `/adapt`.

**User mode** — who you are and how to work with you. Lives in `.dotagent/rules/whoami.md`, always loaded as an always-on rule.

They compose. An `architect` developer in `ship` project mode doesn't produce friction — the agent ships fast and leaves legible debt notes, preemptively, without being asked.

See [SPEC.md](SPEC.md) for the full mental modes design.

---

## Developer Profile

`whoami.md` is the most important file in `.dotagent`. It tells the agent who is operating it — capability level, reasoning style, tone contract, how to handle pushback, what a good work product looks like.

Build it two ways:

**`/whoami`** — set up from a preset (`architect`, `pragmatist`, `explorer`, `learner`) then answer targeted questions.

**`/reflect`** — run this at the end of any conversation where something was corrected, rejected, or reframed. The agent surfaces what it learned about you and appends it directly to `whoami.md`.

> Smooth conversations reveal almost nothing. Friction is the signal. Correction, rejection, reframing — these are the profile building blocks.

The underlying reflection prompt works in any AI agent, with or without dotagent:

> *"Looking at this entire conversation: what did you learn about how I think, what I value, what I reject, or how I work — that if you had known at the start would have changed your first response? Format it as a profile fragment I can add to my working context."*

---

## Project Structure

```
dotagent/
  .dotagent/          # dotagent's own config — eats its own food
  examples/
    rules/            # project-agnostic, reusable anywhere
    skills/
    commands/
    profiles/
    workspaces/       # full stack templates (real runs only)
  COMMUNITY.md        # curated index of community forks
  SPEC.md             # concept and design document
  README.md           # this file
  LICENSE
```

---

## Community & Forks

The `examples/` directory contains only configs that have been personally run and validated. This is not a limitation — it is the quality signal.

**Want to share your stack?** Fork the repo, build your workspace under `.dotagent/`, and open a PR to `COMMUNITY.md` to be listed. You own your fork — your repo, your release cycle, your stack. You can pull updates from the base dotagent repo as the standard evolves.

See [COMMUNITY.md](COMMUNITY.md) for listed community forks.

---

## Design & Spec

The full concept, design decisions, and command specifications are in [SPEC.md](SPEC.md). Read it before contributing. Every decision in there was made deliberately and is explained.

---

## License

[MIT](LICENSE)
