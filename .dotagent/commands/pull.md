---
description: Pull rule and skill updates from the upstream source defined in manifest.yaml.
tags: [pull, sync, upstream]
status: future
---

# /pull

> **Not yet implemented.** This command is a stub for a planned future capability.

Updates rules and skills from the `source` defined in `manifest.yaml`. Pulls from the
upstream repository, shows a diff, and lets the developer accept, reject, or partially
apply changes. Treats agent config as a managed dependency.

## Notes

This command is not implemented in v1. The `source` field in `manifest.yaml` is reserved
for this purpose. Do not attempt to run `/pull` — it will have no effect until implemented.
