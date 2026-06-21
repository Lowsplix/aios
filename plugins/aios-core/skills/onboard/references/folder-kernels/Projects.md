# Projects (the active work)

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

Projects are living folders that grow over time, not flat README files. Every project starts as a single `README.md`, and subfolders are born only when content that needs them arrives. This is where work in motion lives: clients, products, initiatives. Whatever is written here for you is written in Hebrew.

## Routing inside a project

When project information arrives, route it by type:

| Content type | Routes to |
| --- | --- |
| Status update, review, deadline | `Projects/{name}/README.md` |
| Research finding, competitor analysis | `Projects/{name}/research/{topic}.md` |
| Spec, requirement, brief | `Projects/{name}/specs/{name}.md` |
| Draft, script, written content | `Projects/{name}/drafts/{name}.md` |
| Idea, brainstorm | `Projects/{name}/ideas/{name}.md` |
| Working notes, scratch notes | `Projects/{name}/notes/{name}.md` |
| Feedback, review comments | `Projects/{name}/feedback/{name}.md` |

## Rules

- **Subfolders on demand**: do not create empty folders up front. When content that needs a subfolder arrives, create it, write the file, and update the README to point at it.
- **README as index**: the `README.md` is the entry point: overview, status, next steps, and links to content in the subfolders. Do not cram all the project information into it, link outward.
- **Lifecycle**: a new project is just a `README.md`. Subfolders appear as content types are born. A finished project moves to `Intelligence/archive/{name}/`.
- Include `project:` in the frontmatter of every note tied to a specific project.
- Use `[[wikilinks]]` for every person, company, and project mentioned.

## Tie to goals

Every project should connect to a goal from `Context/strategy.md`. In the weekly review, check which goal has no active project (it is drifting), and which project does not serve any goal (it may be time to close it). Tasks connect to projects through `task-list/Tasks.md`.
