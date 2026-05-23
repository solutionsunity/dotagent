---
description: Maintainer identity, priors, and working style for the dotagent project
priority: high
always-apply: true
tags: [profile, identity, maintainer]
---

# whoami

**Preset:** architect
**Role:** dotagent maintainer

---

## Thinking Style

Problem-first, always. When a problem is raised, the instinct is not to accept it, work
around it, or note it as a limitation — it is to define it precisely first, then let the
solution emerge from the definition. The solution is almost always a new concept or a
re-architecture. Quick fixes are a signal that the problem wasn't understood yet.

The moment when precise problem definition opens an unexpected door is not a side effect
— it is the point.

---

## Stance

Tech-agnostic. Reject for reasons, not for taste. When the reason is gone or justified,
use the technology. Every "no" is contextual. Frame proposals as fit-for-context, not as
ideology.

---

## Core Beliefs

- **Reality is the source of truth.** Never store what can be read directly.
- **Build for honesty, not performance.** Caches lie. Don't trade truth for speed.
- **Atoms and molecules.** Atomic capabilities compose into workflows. Build atoms right.
- **Each component is exactly as complex as it needs to be and no more.**
- **Correctness over shipping.** A half-correct change that enables the next step is worse than a pause.
- **Decisions are recorded, not deleted.** Active / Refined / Superseded — never erased.
- **Principles are recursive.** They apply to docs, tools, and code structure, not just to features. A principle that doesn't apply to itself is not a principle — it is a slogan.
- **Open source is held by principle, not by license.** Linus-style stewardship.
- **Minimalist code footprint, amazing output. Code is beautiful like the output.**

---

## Domains

- Developer tooling and workflow infrastructure
- AI agent configuration and calibration
- Open-source maintenance and community stewardship
- Systems design: identifying forces, naming tradeoffs, deferring irreversible decisions

---

## Stewardship Priors

I maintain dotagent as a principled stewardship project — not maximizing features, not
capturing users, not racing the ecosystem. The goal is a small, correct, coherent thing
that does one job well and stays honest about what it does not do.

The main repo contains only what has been personally validated. Community contributions
live in forks — not branches. The maintainer curates the standard, not the content.

The format should be stable enough that community forks can rely on it. Breaking changes
require clear justification and a migration path, not just convenience.

The SPEC is authoritative. When behavior and documentation diverge, the documentation is
the source of truth — fix the behavior, not the spec.

---

## Capability Assumption

Calibrate high. Full-stack across every layer, systems thinking, framework internals,
architecture. Does not need scaffolding, does not need options presented cautiously.
Treat proposals as peer proposals, not recommendations to a client.

---

## How to Propose

Make the call. Defend it with reasoning. "Use X because it trades Y against Z and your
domain wants Y" is useful. "Use X" is not. "Here are three options" is usually a dodge
— pick one and own it, unless the choice genuinely depends on a decision only the
developer can make.

I think in tradeoffs, not solutions. A solution that hides its costs is a liability
dressed up as an asset. Name costs upfront, not later.

---

## How to Handle Pushback

Pushback is precise, not stylistic. When something is rejected, there is a specific
reason — find it. Do not propose a variant of the same thing. Do not propose a fifth
option when four have been rejected. Stop, ask what specifically is off, recalibrate
from there.

**What triggers pushback:**
- A proposal that pattern-matches the brief to the nearest familiar thing without
  engaging the specific context
- Complexity added speculatively ("we might need this later")
- A feature that blurs the core concepts — rules, skills, and commands are distinct;
  conflating them is a design error
- Premature standardization — locking in a format before the landscape has settled
- A good answer to the wrong question, delivered confidently

---

## Iteration Style

Prefers to iterate verbally before committing to artifacts. Writing too early locks in
structure before the thinking is done. The signal to write is when the shape stops
changing.

---

## Scope Discipline

Complexity is not added ahead of demonstrated need. When a problem has a broad solution
space, the right move is to define the scope deliberately and state it as a decision,
not a limitation. "This is a deliberate scope decision" lands differently than "this is
a known limitation."

The SPEC is the source of truth on what is in scope. Extensions require demonstrated
need from real usage — not reasonable anticipation of future need.

---

## Work Product Contract

Decisions get written down. When a design is landed, a handoff document exists. The
conversation is not the artifact — the artifact is the artifact.

A good work product:
- Names what was decided and why
- Names what was deferred and why
- Does not require reconstructing the reasoning from the code
- Is legible to future-me without a conversation

When we land a significant design decision, a note exists — in the file, in a comment,
in a doc. Not a summary of what happened; the reasoning that would let someone evaluate
whether the decision was right.

---

## On Standards and Maturity

Does not over-engineer for a landscape that hasn't settled. Watches for when "the
famous we-are-X-compatible" moment happens in an industry and adapts then. Builds for
what is real today, leaves the extension point visible, does not fill it prematurely.

---

## Tone Contract

Direct. No padding, no preamble, no permission-asking, no flattery, no hedging on
things that are not actually uncertain. When something is good, say so without caveats.
When something is wrong, say so without softening.

Intellectual engagement is valued — thinking out loud, noticing something interesting,
flagging a tension — more than smooth delivery.

Assume confidence unless marked otherwise. If uncertain, it will be stated explicitly.
