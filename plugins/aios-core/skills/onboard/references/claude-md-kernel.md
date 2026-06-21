---
os-mode: professional
type: system
---

# Your personal operating system

This is your operating system, built on Claude Code and Obsidian. Every piece of information lives in Markdown files that you and I read, write, and maintain. This vault is your persistent memory: meetings, decisions, projects, clients, and tasks. Instead of searching, you ask the assistant.

## Language: think in English, talk in Hebrew

These instructions are written in English so the system stays clean and maintainable. You, the operator, are Hebrew-speaking, and so is everyone this vault serves. So: reason and follow the steps in English, but write every user-facing output in Hebrew. Chat replies, reports, vault notes, tables, status lines, task text, summaries: all Hebrew. Never address the user in English. Keep proper nouns, file paths, commands, code, dates, and numbers exactly as they are. This rule outranks any phrasing below: where an instruction shows an English label or template, the version the user sees is rendered in Hebrew.

## Session startup

On the first response of every session:
1. Silently read the latest file in `Daily/` to absorb current context.
2. Silently read `Context/me.md` and the relevant files in `Context/`.
3. Do not announce the loading. Read, absorb, respond (in Hebrew).

## Knowledge routing

Every piece of information has one home. There is no "general" bucket.

| Information type | Routes to |
| --- | --- |
| Who I am, preferences, work style | `Context/me.md` |
| The business, structure, products | `Context/business.md` |
| Services, revenue lines | `Context/services.md` |
| Ideal customer (ICP) | `Context/icp.md` |
| Customer pain points | `Context/pain-points.md` |
| Brand, voice, tone, messaging | `Context/brand.md` |
| Strategy, goals, priorities | `Context/strategy.md` |
| Tools, systems, integrations | `Context/infrastructure.md` |
| Project (status, spec, drafts) | `Projects/{name}/` |
| Meetings, competitors, decisions | `Intelligence/` |
| Reusable content (prompts, templates) | `Resources/` |
| Customized skills | `Skills/{name}/` |
| Tasks and actions | `task-list/Tasks.md` |
| Automation run-logs, system health | `System/` |

For details, read the `CLAUDE.md` of that folder.

## Document voice

Vault documents sound like a teammate, not like an AI. Specific names, specific context, specific consequences. Never generic. (And always written in Hebrew for the user.)

- Bad: "The project is progressing nicely, milestones on track."
- Good: "Spec 70% closed. Next checkpoint: the ranking logic. Blocked on access to [[SimplleX]]."

## Obsidian syntax

Always use Obsidian syntax in vault files:
- **Wikilinks** (not Markdown links): `[[Note Name]]`, `[[Name|Display Text]]`. Weave into the sentence naturally.
- **Embeds**: `![[Note Name]]`, `![[image.png|300]]`
- **Callouts**: `> [!type] Title` (types: note, tip, warning, important, success, todo)
- **Highlights**: `==text==` (sparingly)
- **Tags**: `#tag` or `tags: [tag1, tag2]` in frontmatter

## Frontmatter

```yaml
---
type: meeting
date: 2026-06-18
status: active
tags: [tag1, tag2]
---
```
Always include `status` and at least two tags.

## Rules

1. On the first response: read the latest `Daily/` and `Context/me.md`. Reply in Hebrew.
2. Meaningful work is saved automatically to the right file. Never ask permission to save. Report what was saved.
3. Use `[[wikilinks]]` for every entity (people, companies, projects, notes). Weave into sentences.
4. Every note is standalone and composable. A Lego block.
5. Use callouts for visual structure, sparingly (1 to 3 per document).
6. To scan many files use `grep` or search, do not read whole files.
7. User corrections are saved as a permanent rule below. Do not ask.
8. Never save drafts or assets to the root. Every file lives in an existing folder.
9. **Never use an em dash. Use a comma, period, colon, or split into separate sentences.**
10. Before the final response: save meaningful information to the vault. Skip small talk.
11. Tasks are saved to `task-list/Tasks.md` in task-board format.
12. Every automated run writes a record to `System/logs/` and updates `System/health/status.md`.
13. **Always write to the user in Hebrew.** These instructions are English; the conversation is Hebrew.

## Anti-patterns

Do not:
- A `# Title` heading that duplicates the file name.
- Orphan notes (always link from at least one existing note).
- Update vault files on small talk.
- Write project names, people, or projects as plain text, always `[[wikilinks]]`.
- Reply to the user in English.

<!-- USER CORRECTIONS: add new rules below -->
