# FAQ

---

## Modes

**What mode should I use for a brand new project?**

`architecture`. The project has no authoritative codebase yet. Documentation is being
written as the source of truth — code follows from it. The agent treats any
implementation as provisional and pushes decisions back to the doc before they land.

---

**My codebase works but is messy. Docs are solid. What mode?**

`refactor`. The source of truth already exists. The agent treats docs as authoritative,
flags code that contradicts them, and restructures toward the known target. If you have
no formal docs, add a `mode-note` describing the target state — the agent will ask about
intent before restructuring rather than assume the existing code is correct.

---

**What is the difference between `refactor` and `maintain`?**

`maintain` means don't improve — match existing patterns, consistency over cleverness.
Use it when the codebase is stable and deviating from its patterns is the risk.
`refactor` means actively move toward a known better state. The existing patterns are
what you are moving away from, not what you are matching.

---

**When is `migrate` the right choice?**

Only when infrastructure, data, or a platform substrate is moving from A to B — database
schema migrations, cloud provider switches, platform rewrites where data continuity is
the priority. Not for code restructuring (`refactor`) and not for documentation work
(`architecture` or `refactor`).

---

**What is the difference between `explore` and `architecture`?**

`explore` means no target exists yet — the agent proposes, thinks out loud, keeps
options open, avoids commitment. `architecture` means you are building toward a defined
target — the document is the source of truth and code will follow. `explore` is
pre-direction. `architecture` is post-direction, pre-implementation.

---

**My repo is a collection of modules — Odoo, monorepo, or similar. Each module is in a
different phase. What mode?**

`scoped`. Set `mode: scoped` in `manifest.yaml`. The agent asks which module you are
working in at the start of every session, then applies the stated mode for that session.
Same behavioral depth as a globally declared mode — just session-declared instead of
manifest-declared.

---

**Can I switch modes mid-project?**

Yes — that is what `/adapt` is for. Run it when the project phase changes. It surfaces
stale rules or skills and asks you to confirm the current mode. The agent's behavior
shifts as soon as the manifest is updated.

---

**I am in `ship` mode. Does the agent stop reasoning architecturally?**

No. Mode shifts the optimization target, not the agent's capability. An `architect` user
in `ship` mode still thinks architecturally — what changes is when and how that thinking
surfaces. The agent ships fast and leaves legible debt notes rather than stopping to
refactor. The tradeoffs are named and deferred visibly, not ignored.

---

## Setup

**Do I need to install anything?**

No. Open your AI agent in your project directory and paste the bootstrap prompt from the
README. The agent reads the init command and walks you through setup. Your `.dotagent/`
is built from the conversation.

---

**I already have a `.claude/` directory. Will `/init` overwrite it?**

No. `/init` detects existing config and offers two choices: Adopt (wrap it without moving
anything) or Migrate (move it into `.dotagent/` with managed symlinks). Both are complete
outcomes — no half-states, no per-file deferral.

---

**What is the difference between a rule, a skill, and a command?**

Rules are always-on — the agent applies them without being asked. Hard constraints:
things that must always or never happen.

Skills are procedures — named sequences the agent knows how to perform in this workspace
specifically. How tests are run here, how a deployment is triggered here.

Commands are the public interface — slash commands you type directly. They compose rules
and skills into developer-facing workflows.

---

**Where does `whoami.md` live and why is it a rule?**

It lives in `.dotagent/rules/whoami.md`. It is a rule because it is always-on — the
agent loads it as part of its standing context, not as something you invoke. Your
identity, capability level, tone contract, and rejection patterns should be present in
every interaction, not just when you remember to mention them.

---

## Community

**I want to share my stack config. What do I do?**

Fork the repo. Build your workspace in your fork's `.dotagent/`. Then open an issue
linking to your fork to be considered for listing in `COMMUNITY.md`. You own your fork —
your repo, your release cycle, your stack. The main repo stays clean.

---

**What does a fork need to be listed in `COMMUNITY.md`?**

A valid `manifest.yaml` with `stack` and `maintained-by` declared,
`dotagent-compatible: true` set, and traces that are real runs — not invented. Atomic
examples should carry `stack: agnostic` in frontmatter to signal reusability.
