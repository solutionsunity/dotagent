---
description: Set up or update rules/whoami.md. Starts from a preset, asks targeted questions to move from preset to personal, writes rules/whoami.md on confirmation.
tags: [whoami, identity, setup]
---

# /dotagent-whoami

Sets up or updates the developer profile. Can be run standalone at any time, or is
invoked automatically during `/dotagent-apply` when no profile exists. Starts from a preset
starting point, then asks targeted questions to move from preset shape to personal
substance.

## Trace

```
/dotagent-whoami

> "Starting from a preset or from scratch?"
  [ ] explorer   [ ] pragmatist   [ ] architect
  [ ] learner    [ ] scratch

User: architect

Agent seeds the profile with the architect preset, then asks
targeted questions to move from preset to personal:

> "What domains do you work in?"
User: full-stack, systems, Odoo internals, Arabic-first products

> "How should the agent handle proposals you push back on?"
User: my pushback points at something specific — figure out
  what, don't propose a different brand of the same thing

> "What does a good work product look like to you?"
User: decisions written down, handoff doc when we land a design

Agent previews the generated rules/whoami.md and shows
the full content before writing:

  rules/whoami.md                         [new]

> "Write this file? [yes / no / edit]"

User: yes
```

## Notes

- Presets are starting points only — they seed the shape of the profile. The developer
  authors the substance. A real profile is a calibration document, not a preference list.
- The most accurate profile does not come from answering abstract setup questions — it
  comes from real interactions. Use `/dotagent-reflect` after real conversations to build on what
  `/dotagent-whoami` seeds.
- `/dotagent-whoami` never overwrites an existing `rules/whoami.md` without showing a diff and
  asking for confirmation. The confirmation contract is `[yes / no / edit]`.
