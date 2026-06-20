---
description: Re-read every rules file verbatim, restate the binding constraints found in them, and prosecute recent turns for drift. Recalibration for long sessions where agent-side compaction has paraphrased the rules.
tags: [refresh, calibration, drift, rules]
---

# /dotagent-refresh

Re-grounds the agent in its own rules. Long sessions are compacted **agent-side** to
keep the context window workable — and compaction paraphrases the rules like ordinary
context, silently degrading the very function that judges the agent's output. Load-bearing
clauses are exactly what summarization smooths into a vague "follow the rules." This
command reloads them verbatim and proves they are loaded.

It is the inverse of `/dotagent-reflect`: reflect mines the session and writes to the
rules; refresh reads the rules back into the agent.

A re-read alone is theater. `/dotagent-refresh` re-reads, then **prosecutes**:

1. **Reload** every file in `rules/` verbatim — not the summary in context, the file on
   disk.
2. **Restate** the binding constraints found in the loaded files, in the agent's own
   words — to prove they are loaded, not merely present. If no explicit behavior rules
   exist, say so.
3. **Audit** the recent turns against them and report drift found, or state "none found"
   honestly. Naming a violation that already shipped is the point, not avoiding it.

## Trace

```
/dotagent-refresh

Agent reloads rules/ from disk and restates the constraints it is
most prone to eroding:

  Reloaded: rules/whoami.md, rules/commit-style.md

  Holding (from loaded files):
  - whoami.md § Pushback — do not propose a variant of the same
    thing; stop and ask what specifically is off.
  - whoami.md § Tone Contract — no hedging on things that are not
    actually uncertain.
  - commit-style.md — Conventional Commits required; no merge commits.

  Auditing last 6 turns:
  ✗ Turn 4 — proposed a variant after pushback on the first approach.
    whoami.md § Pushback violation. Corrected in turn 5.
  ✓ Turns 1-3, 5-6 — no drift found.

> "1 drift found, already corrected. Holding the line from here."
```

## Notes

- Run `/dotagent-refresh` when the agent starts asserting without citing, proposing
  variants after a rejection, or reaching for "best practices" — the smell of compacted
  rules.
- The command reads the **files on disk**, deliberately bypassing the possibly-paraphrased
  copy in the context window. If a rule on disk and the agent's stated understanding
  disagree, the disk wins and the delta is surfaced.
- The restate step is agent-driven from the loaded files — it does not assume any
  specific named constructs. What the rules contain is what gets restated.
- The honest output is the value. "No drift found" is only valid after an actual
  line-by-line pass — not a reflex. A violation is named with its turn and the rule it
  broke, not softened.
- This is a mitigation, not a root fix. The root fix — rules that survive compaction
  verbatim — is agent-platform behavior. `/dotagent-refresh` is the deep recalibration
  until that exists.
