# AIOS-CORE BUILD SPEC (shared contract for builder agents)

You are building one or more skills inside the `aios-core` Claude Code plugin: a Hebrew, client-facing AI Operating System modeled on Ben's `os-setup` plugin, simplified. Read this whole file before writing. Match these conventions exactly so all skills are consistent.

## Repo layout (already scaffolded)
```
~/repos/aios/
├── .claude-plugin/marketplace.json        (done)
├── BUILD-SPEC.md                          (this file)
└── plugins/aios-core/
    ├── .claude-plugin/plugin.json         (done)
    └── skills/
        ├── onboard/{SKILL.md, references/}
        ├── audit/{SKILL.md, references/}
        ├── level-up/SKILL.md
        ├── morning-report/{SKILL.md, references/}
        ├── supervisor/SKILL.md
        ├── connect/SKILL.md
        └── doctor/SKILL.md
```
Write ONLY the skill folder(s) assigned to you. Use absolute paths under `/Users/adir/repos/aios/`.

## Language + voice (non-negotiable)
- **All user-facing skill content is in HEBREW.** Simple, clear, warm-professional. Talk to a busy solo business owner, not an engineer.
- **NEVER use em dashes (—).** Use commas, periods, colons, parentheses, or split sentences. Hebrew and English alike.
- Keep it SIMPLE. Short steps. No jargon dumps. The client should never feel lost. Prefer plain-language over "absolute path", "OAuth scope", etc. (explain in plain Hebrew when unavoidable).
- Obsidian-native syntax in any vault file the skill writes: wikilinks `[[שם]]`, callouts `> [!type]`, frontmatter YAML.

## SKILL.md conventions (copy Ben's exactly)
- Frontmatter has EXACTLY two keys:
  ```yaml
  ---
  name: <kebab-case-slug, matches folder, becomes the /command>
  description: <ONE dense paragraph. What it does + concrete behaviors + embedded trigger phrases in BOTH Hebrew and English. e.g. "Use when user says 'audit', 'ביקורת', or runs /audit.">
  ---
  ```
  No other frontmatter keys.
- Body: `# Hebrew Title` (human name, not slug). Then a `> [!note]` one-liner of what it does. Then `## שלב N` numbered phases/steps executed in order. Use Markdown tables for any mapping. End every operational skill with `## חוקים` (hard rules, terse imperatives) and where useful `## פתרון תקלות`.
- Reference files live in the skill's `references/` sibling folder. Read paths resolve relative to the SKILL.md dir; use `${CLAUDE_PLUGIN_ROOT}` when a skill must locate its own bundled assets across installs, with a fallback note.
- Inline bash for real actions (mkdir, gws calls, wacli calls) in fenced ```bash blocks, exactly like Ben's os-setup Phase A and os-mcp.

## The standard vault the OS lives in (created by /onboard)
Directory tree onboard must create (professional/solo mode):
```bash
mkdir -p .claude Context Projects Daily Resources Skills \
  Intelligence/meetings Intelligence/decisions Intelligence/archive \
  System/logs System/health
```
- `Context/` holds the brain: me.md, business.md, services.md, icp.md, brand.md, strategy.md, infrastructure.md, pain-points.md (Hebrew filenames are fine too, but keep these English slugs for routing consistency).
- `System/` is the OBSERVABILITY home: `System/logs/` (one dated run log per routine run), `System/health/status.md` (last-run + connection state). Every routine and the supervisor APPEND a run record here. This is how Adir and the client know what ran and what broke.
- Root `CLAUDE.md` kernel is provided at `plugins/aios-core/skills/onboard/references/claude-md-kernel.md` (Hebrew). onboard copies it to the vault root and fills `os-mode`.

## Run-logging convention (ALL routines + supervisor use this)
Every scheduled or triggered run appends a record to `System/logs/YYYY-MM-DD.md` and updates `System/health/status.md`. Record format (Hebrew labels):
```
- HH:MM | <skill-name> | ✅ הצליח / ⚠️ חלקי / ❌ נכשל | <one line: what ran, counts, or the error> | משך: <sec>
```
`System/health/status.md` keeps a small table: per skill, last-run timestamp + last status + next-scheduled. This is the single pane the `/doctor` skill reads.

## Frontmatter standard for vault notes the skills write
```yaml
---
type: <note|meeting|project|report|health>
date: YYYY-MM-DD
status: <active|completed>
tags: [tag1, tag2]
---
```

## Scheduling
Routines (morning-report) are wired with the user's `schedule` skill / cron. The skill should INSTRUCT how to schedule itself (a `## תזמון` section) and write the trigger, not assume it runs itself. Note clearly that the machine/terminal must be on at the scheduled time (design is outcome-push during the client's working hours, not a 24/7 server).

## Self-verify before returning
After writing, `ls -R` your skill folder and `cat` each SKILL.md you wrote to confirm it is valid. Return a short summary: files written (absolute paths), what each skill does, and any assumption you made.
