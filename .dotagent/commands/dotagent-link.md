---
description: Create or repair symlinks from .dotagent/ into each active agent's expected directory. Reads active targets from manifest.yaml. Safe to run any time.
tags: [link, symlinks, manifest]
---

# /dotagent-link

Creates or repairs symlinks from `.dotagent/` into each active agent's expected
directory. Reads active agents and targets from `manifest.yaml`. Safe to run at any
time — it checks current state before acting and only creates what is missing.

## Trace

```
/dotagent-link

  Reading manifest.yaml — agents: claude, augment — mode: solid

  .claude/rules    →  .dotagent/rules     ✓ exists
  .claude/skills   →  .dotagent/skills    ✗ missing — created
  .claude/commands →  .dotagent/commands  ✓ exists
  .augment/rules   →  .dotagent/rules     ✓ exists
  .augment/skills  →  .dotagent/skills    ✓ exists

  Done. 1 link created.
```

## Linking Behavior by Platform

| Platform | Mechanism            | Notes                                        |
|----------|----------------------|----------------------------------------------|
| Linux    | Symlinks (`ln -s`)   | Native, no elevation required                |
| macOS    | Symlinks (`ln -s`)   | Native, no elevation required                |
| Windows  | Directory junctions  | No elevation required; directory-level links |
| Windows  | `.lnk` shortcuts     | File-level links where junctions don't apply |

## Copy Fallback

In rare environments where linking fails, `/dotagent-link` stops and asks explicitly:

```
  Linking failed — symlinks not supported in this environment.

  Copy mode available as fallback. Files will be copied instead
  of linked. Changes to .dotagent require re-running /dotagent-link
  to propagate.

  Which agents should copy mode apply to?
  [ ] Claude only   [ ] Augment only   [ ] Both

> "Proceed with copy mode? [yes / no]"
```

Copy mode is never automatic. The developer decides scope explicitly.

## Notes

- Run `/dotagent-link` after editing `manifest.yaml`, after adding new agents, or after
  any manual change to the `.dotagent` directory structure.
- All paths in `manifest.yaml` are relative to the repository root.
- If a link target directory does not exist inside `.dotagent/`, `/dotagent-link` reports
  it and does not create a broken link.
