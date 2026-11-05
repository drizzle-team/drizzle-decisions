# Drizzle decision records

This repository holds the architecture decision records of [drizzle-orm](https://github.com/drizzle-team/drizzle-orm). A record says what the team decided, why, and which options it did not take. Coding agents read the records before they change the code, and people can read them to learn why the code works the way it does.

## Layout

Each folder holds the records of one part of the drizzle-orm repository:

- `drizzle-orm/`: the `drizzle-orm` package and decisions that affect the whole repository.
- `drizzle-kit/`: the `drizzle-kit` package.
- `postgres/`: the Drizzle Postgres driver.

A record is one Markdown file named `NNNN-short-title.md`. Each folder has its own numbers. A record names another record of the same folder as `ADR NNNN`. A record in another folder is named by its folder and number, for example `drizzle-kit/0005`.

## Add a record

1. Take the next free number in the folder.
2. Start with a `# Title` that states the decision as a sentence. The first paragraphs say what was decided and why.
3. Add `## Considered options` with the options that the team did not take and the reason for each.
4. Add `## Consequences` with what changes because of the decision.
5. Use plain Markdown links. Obsidian also understands them.
6. Open a pull request.

Two branches can take the same number. The `check` workflow fails when two records of one folder share a number. If it fails after you update your branch, give your record the next free number.

Do not change the decision of an existing record. When a new decision replaces a record, the new record says which record it replaces, and the old record gets this front matter:

```markdown
---
status: superseded by ADR-NNNN
---
```
