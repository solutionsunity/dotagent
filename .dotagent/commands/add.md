---
description: Scaffold a new skill or rule file interactively. Asks targeted questions, previews the file, and writes on confirmation.
tags: [add, scaffold, skill, rule]
---

# /add

Scaffold a new skill or rule file interactively. The agent asks targeted questions,
generates the file with correct frontmatter and structure, and shows a preview before
saving. Accepts `skill` or `rule` as the type argument.

## Trace — `/add skill`

```
/add skill migrate-database

> "What does this skill do?"
User: runs pending migrations and prints a summary

> "How is it triggered in this repo?"
User: `npm run db:migrate`

> "Any preconditions?"
User: requires DATABASE_URL to be set

Agent previews the file:

  .dotagent/skills/migrate-database.md          [new]

  ---
  description: Run pending database migrations and print a summary
  tags: [database, migrations]
  ---

  # migrate-database

  Run pending migrations with `npm run db:migrate`.

  **Preconditions:** `DATABASE_URL` must be set in the environment.

> "Write this file? [yes / no / edit]"
```

## Trace — `/add rule`

```
/add rule no-console-log

> "What should this rule enforce?"
User: no console.log in production code — use the logger service

> "Does this apply everywhere or only in specific directories?"
User: everywhere except test files

Agent previews the file:

  .dotagent/rules/no-console-log.md             [new]

> "Write this file? [yes / no / edit]"
```

## Notes

- If a file with the same name already exists, `/add` stops before asking any
  questions and confirms whether to overwrite. It never silently replaces existing
  config. The confirmation contract is `[yes / no]` at that gate — the rest of the
  flow only proceeds on confirmation.
- Generated files include correct YAML frontmatter for both Claude and Augment
  compatibility. Augment reads and uses the frontmatter; Claude ignores it without
  breakage.
- Valid types: `skill`, `rule`. Commands have their own format — use the command
  template from SPEC.md section 9 for those.
