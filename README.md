# AIOS — Adir Sellam client AI Operating Systems

A Claude Code plugin marketplace for delivering Hebrew AI operating systems to clients. One base plugin installed for every client, plus per-client vertical plugins. Built on Claude Code + Obsidian, modeled on Ben's `os-setup`, simplified and in Hebrew.

## Layout
```
aios/                                  ← marketplace repo (this, GitHub: Lowsplix/aios)
├── .claude-plugin/marketplace.json    ← lists the plugins
└── plugins/
    └── aios-core/                      ← THE STANDARD LAYER (install for every client)
        ├── .claude-plugin/plugin.json
        └── skills/
            ├── onboard/        /onboard      הקמה: bootstrap vault + Hebrew interview + write Context
            ├── connect/        /connect      חיבור: wire Gmail/Drive/Calendar (gws), WhatsApp (wacli), API keys
            ├── audit/          /audit        ביקורת: score the 4 C's (Context/Connections/Capabilities/Cadence) + gaps
            ├── level-up/       /level-up     שדרוג: find the next skill to build (5 questions) + scaffold it
            ├── morning-report/ /morning-report  דוח בוקר: daily Hebrew brief
            ├── supervisor/     /supervisor   מפקח: query/command the system over WhatsApp
            ├── doctor/         /doctor       בדיקה: health check of connections + routines
            ├── update/         /update       עדכון: pull latest plugin (self-update, no client action)
            └── health-ping/    /health-ping  פינג בריאות: daily heartbeat to Adir (fleet view)
```
Per-client vertical plugins (e.g. a real-estate `aios-realestate` with deal-sourcing/ranking/owner-id) get added under `plugins/` and listed in the marketplace later.

## Install (per client)
```
/plugin marketplace add Lowsplix/aios
/plugin install aios-core@aios
```
Then in an empty folder (the future vault): `/onboard`. After that: `/connect`, then the rest.

## Concern 1: updating client skills without the client doing anything
Because the skills ship as ONE git-backed plugin, you maintain a single source of truth:
- **You** improve a skill (e.g. fix it for one client) → bump `plugins/aios-core/.claude-plugin/plugin.json` `version` → push to this repo.
- **Every client** pulls it. The `/update` skill does `git pull` on the client's plugin clone and reports `old -> new` version. Wire `/update` as a **daily scheduled routine** in each client vault and the system self-updates overnight, no client action. You push once, everyone is current by morning.
- The "export this and give it to my other client" flywheel (Liam) is exactly this: cross-client learnings land in `aios-core`; verticals stay in per-client plugins.
- **Who is on what version:** `/doctor` and `/health-ping` report the installed version, so you always know the fleet's state.
- Note: confirm the exact Claude Code `/plugin` refresh command for your version; the `git pull` on the marketplace clone is the pragmatic mechanism and is what `/update` automates.

## Concern 2: observability (know what works, what breaks, stay in control)
Everything routes through the `System/` folder in each client vault:
- **`System/logs/YYYY-MM-DD.md`** — every routine + supervisor APPENDS a run record: `HH:MM | skill | ✅/⚠️/❌ | what ran or the error | משך`.
- **`System/health/status.md`** — a small table: per connection + per routine, last-run + last-status + version. The single pane.
- **`/doctor`** — actively probes connections (`gws` auth, `wacli doctor`) and each routine's last run, prints a Hebrew ✅/⚠️/❌ table with the fix. The client's and your on-demand health surface.
- **morning-report** carries a one-line health line, so the client sees system health daily without asking.
- **`/health-ping`** — a daily heartbeat that sends ONLY the health summary line to you (WhatsApp/Sheet), so you get a fleet view across all clients without logging into each machine. Health summary only, never business content, client-consented via `System/health/maintainer.md`.

You own the canonical plugin source; the client runs a versioned, logged copy; `/doctor` + `/health-ping` keep you in control of state across the fleet.

## Conventions
See `BUILD-SPEC.md`. Hebrew, simple, no em dashes. SKILL.md frontmatter is exactly `name` + `description` (triggers embedded in the description). Skills auto-discover, no registry to maintain.
