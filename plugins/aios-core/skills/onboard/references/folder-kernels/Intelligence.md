# Intelligence (the organizational knowledge)

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

This is where knowledge that accumulates from the world lives: meetings, decisions with their reasoning, competitors, market intelligence, and the archive. It is the long-term memory of the system. When you ask "what did we agree in that meeting?" or "why did we decide this way?", the answer lives here. Whatever is written here for you is written in Hebrew.

## Routing

| Category | Type | Routes to |
| --- | --- | --- |
| Meetings | Client call | `meetings/client-calls/{name}/` |
| Meetings | One-on-one | `meetings/one-on-ones/` |
| Meetings | Anything else | `meetings/general/` |
| Knowledge | Competitor insight | `competitors/{name}.md` |
| Knowledge | Market intelligence | `market/{topic}.md` |
| Knowledge | Opportunity analysis (a product you are considering building) | `opportunities/{name}-teardown.md` |
| Knowledge | Decision with reasoning | `decisions/YYYY-MM-DD-{title}.md` |
| Knowledge | Recurring process or procedure | `processes/{name}.md` |
| Archive | Content that is finished or frozen | `archive/{name}/` |

(Folders are created on demand. At setup, `meetings/`, `decisions/`, and `archive/` exist. The rest are born when content arrives.)

## Decision records

Every decision is saved with the **why**, with specific context: who was involved, what tension led to it, and which tradeoff was chosen. Use `> [!important]` for the core decision statement, and `> [!warning]` for a known risk or cost. Without the why, the decision is worthless in half a year.

## Rules

- A meeting goes under `meetings/` by its type, with frontmatter `type: meeting`, `date:`, and the participants as `[[wikilinks]]`.
- Move finished projects, competitors that are no longer relevant, and stale research to `archive/`.
- Every note here stands on its own and is linked to at least one other entity with a `[[link]]`.
- Do not write a SOP for a specific department here if there are no departments. In a solo business, recurring processes go to `processes/`.
