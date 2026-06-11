---
description: Run the reflection prompt against the current conversation and offer to append profile fragments directly to rules/whoami.md.
tags: [reflect, profile, calibration]
---

# /dotagent-reflect

Runs the reflection prompt against the current conversation and surfaces profile
fragments — things the agent learned about how the developer thinks, what they value,
what they reject, or how they work. Offers to append the output directly to
`rules/whoami.md`. The mechanism for keeping the profile accurate over time.

## Trace

```
/dotagent-reflect

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

User: edit

Agent presents each fragment for inline editing before appending.
Developer refines Fragment 2, accepts Fragment 1 as-is.

Fragments appended to rules/whoami.md.
```

## The Underlying Prompt

The reflection prompt — usable in any AI agent, with or without dotagent:

> *"Looking at this entire conversation: what did you learn about how I think, what I
> value, what I reject, or how I work — that if you had known at the start would have
> changed your first response? Format it as a profile fragment I can add to my working
> context."*

The bar is intentionally high — *what would have changed the first response* — which
filters out noise and surfaces only the priors that actually matter.

## Notes

- Run `/dotagent-reflect` after conversations where something was corrected, rejected, or
  reframed. Smooth conversations reveal almost nothing. Friction is the signal.
- The underlying prompt is agent-agnostic. It works in Claude, Augment, or any LLM
  interface — no dotagent infrastructure required. The practice is the value; the
  command is a convenience.
- `/dotagent-reflect` does not overwrite existing profile content. It appends. The developer
  reviews before anything is written.
