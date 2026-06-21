# Daily (the journal)

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

This is your daily journal. Every day where something meaningful happens gets one `YYYY-MM-DD.md` file: what you worked on, what was decided, what is still open. This is the first context the system reads at the start of every session, to remember where things stand. Whatever is written here for you is written in Hebrew.

## When it is written

- At the start of every session, the system silently reads the latest file here to absorb current context.
- At the end of meaningful work, a record is added to the current day: what was done, decisions, next steps.
- Small talk does not go into the journal. Only real work.

## Frontmatter

```yaml
---
type: daily-note
date: YYYY-MM-DD
status: active
tags: [daily]
---
```

## Recommended structure

```markdown
## The session
- **Focus**: [what we worked on]
- **Done**: [what was closed]
- **Next steps**: [what is left]
```

## Rules

- One file per day, named `YYYY-MM-DD.md`. Do not create daily files in the vault root, only here.
- Write in the voice of a teammate: specific, with names and `[[wikilinks]]`, not "made nice progress".
- The journal is a summary, not the source of truth. Information that needs a permanent home (a decision, a spec, a customer detail) is also saved to the right folder (`Intelligence/`, `Projects/`, `Context/`), and the journal just links to it.
