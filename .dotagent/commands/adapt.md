---
description: Re-study the workspace and surface drift between what rules and skills assume and what is actually true today. Also the right command when project mode changes phase.
tags: [adapt, drift, mode, maintenance]
---

# /adapt

Re-studies the workspace and identifies drift between what rules and skills currently
assume and what is actually true today. Does not rebuild from scratch — only surfaces
and resolves deltas. Also the right command to run when the project phase changes and
the mode in `manifest.yaml` needs to be updated.

## Trace

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

Agent walks each delta in sequence. Shows the specific line,
proposes the update, waits for confirmation before writing.

> "Current project mode is `ship`. Still accurate?"
User: no — switching to solid now

> "Any context for the change? (optional)"
User: post-launch stability phase

Agent updates manifest.yaml: mode → solid, mode-note written.

  Done. 2 rules updated, manifest.yaml updated.
```

## Notes

- `/adapt` compares rules and skills against the live workspace — it reads actual file
  paths, environment markers, CI config, and installed dependencies.
- "Review each" walks deltas one at a time. The agent shows the specific line that
  triggered the drift signal before proposing any update.
- The agent does not write anything until the developer confirms each delta or the
  batch. The confirmation contract is `[yes / no / review each]` at the batch level,
  then `[yes / no / edit]` for each individual item in review mode.
- Mode check is always included. If the current mode is still accurate, the developer
  says so and the agent moves on — no update written.
- `/adapt` is not a replacement for `/sync`. It surfaces content drift; `/sync` repairs
  structural drift (missing links, stale targets).
