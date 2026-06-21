---
name: audit
description: "Inspects the AIOS vault against the 4 C's (Context, Connections, Capabilities, Cadence) and produces a Hebrew scorecard. Reads Context/, Skills/, System/health/status.md, and the scheduled jobs; scores each C 0-100 with the deciding reason; computes an overall score; ranks the top 3 gaps by leverage with a concrete next action each. Writes the report to System/health/audit-YYYY-MM-DD.md and appends a run-log line. Light, markdown-only, no HTML. Use when the user says 'audit', 'ביקורת', 'תבדוק את המערכת', 'כמה המערכת בנויה', 'ציון למערכת', or runs /audit."
---

# System audit (4C)

> [!note] A snapshot of how built-out your system really is. It gives a score of 0 to 100 for each of the four C's, an overall score, and the three most important gaps with a concrete action for each.

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

The framework is four C's: **Context** (how much of the business is documented), **Connections** (which systems are connected), **Capabilities** (which skills exist), **Cadence** (which routines run on their own). That is the backbone of the audit. The score checks reality against what should be, it does not promise or flatter.

Before you start, read `references/four-c-rubric.md` to know how to score each C (what 0, 50, 100 look like).

## Step 1: Gather the evidence

Collect everything needed to score. Do not read whole files if you can scan. Use `grep` and `ls`.

```bash
# Context: how many Context files exist and how full they are
ls -la Context/ 2>/dev/null
wc -l Context/*.md 2>/dev/null
# Flag empty or template-only files. A real placeholder from the skeleton = single square brackets with Hebrew text inside
# (e.g. "[name, role]"). Be sure not to count frontmatter (tags:), callouts ([!note]), or wikilinks ([[...]]).
grep -rnE '\[[^]!]*[א-ת][^]]*\]' Context/ 2>/dev/null | grep -v 'tags:' | grep -v '\[\[' | grep -v '\[!'

# Capabilities: which skills and subagents exist
ls -d Skills/*/ 2>/dev/null
ls .claude/agents/*.md 2>/dev/null

# Connections and cadence: the latest system state
cat System/health/status.md 2>/dev/null
ls System/logs/ 2>/dev/null | tail -7
```

What to extract from each source:
- **Context**: how many of the 8 core files in `Context/` exist and are really filled (not an empty template with `[brackets]`). The files: `me.md`, `business.md`, `services.md`, `icp.md`, `brand.md`, `strategy.md`, `infrastructure.md`, `pain-points.md`.
- **Connections**: from `System/health/status.md` and `Context/infrastructure.md`, which systems are actually connected: Gmail, Drive, Calendar, WhatsApp, and any additional data sources. A connection counts only if it has evidence (a "connected" status or a recent successful run), not just intent.
- **Capabilities**: how many skills in `Skills/` and subagents in `.claude/agents/` exist and are valid (they have a `SKILL.md`).
- **Cadence**: which scheduled routines run, and when they last ran, per `System/logs/` and `System/health/status.md`. A routine that has not run for a week counts as stuck.

If a folder is missing entirely, that is a finding in itself (a low score for that C), not an error. Continue.

## Step 2: Score each C

Per `references/four-c-rubric.md`, give each C a score of 0 to 100 and one sentence with the **deciding reason** for the score (not a generic description, but what specifically decided it: "5 of 8 context files are empty", "WhatsApp is not connected", "no routine ran this week").

Compute an overall score as a simple average of the four C's, rounded to a whole number.

| Score | Meaning |
| --- | --- |
| 80 to 100 | A built-out system. Run an audit once a month. |
| 60 to 79 | Good base, clear gaps. Handle the three gaps. |
| 40 to 59 | Half a system. Missing build hurts the value. |
| Under 40 | Mostly a skeleton. Needs real building before it works on its own. |

## Step 3: Rank the gaps by leverage

This is the heart of the audit. Do not list everything that is missing. Pick the **three gaps that would give the most return** if closed, and rank them from highest to lowest.

Leverage = how much time or money this gap wastes right now, times how easy it is to close. A gap that blocks three other routines beats a cosmetic gap. Ask: "If I close only this one this week, what gets unblocked?"

For each gap write:
- **The gap**: one sentence, specific. With a `[[wikilink]]` to the entity if relevant.
- **Why it matters**: what it blocks or wastes right now.
- **The next action**: one concrete step you can take, ideally in some skill (e.g. "run `/connect` to connect Gmail", "fill `Context/icp.md`", "schedule `/morning-report`").

## Step 4: Write the report

Write the report to `System/health/audit-YYYY-MM-DD.md` (replace with today's date). Markdown only, no HTML. Render the report content to the user in Hebrew. Keep exactly this structure (the headings and display values are shown in Hebrew because this is the client-facing deliverable):

```markdown
---
type: health
date: YYYY-MM-DD
status: completed
tags: [audit, health, 4c]
---

> [!note] ביקורת 4C מתאריך YYYY-MM-DD. ציון כולל: {ציון}/100.

## ציון 4C

| C | תחום | ציון | הסיבה המכרעת |
| --- | --- | --- | --- |
| הקשר | כמה מהעסק מתועד | {0-100} | {משפט אחד} |
| חיבורים | אילו מערכות מחוברות | {0-100} | {משפט אחד} |
| יכולות | אילו מיומנויות קיימות | {0-100} | {משפט אחד} |
| קצב | אילו שגרות רצות לבד | {0-100} | {משפט אחד} |
| **כולל** | | **{ממוצע}** | {פירוש לפי הטבלה} |

## שלושת הפערים החשובים ביותר (לפי מינוף)

### 1. {הפער}
**למה זה מזיז:** {מה זה חוסם או מבזבז}
**הפעולה הבאה:** {צעד קונקרטי אחד}

### 2. {הפער}
**למה זה מזיז:** {...}
**הפעולה הבאה:** {...}

### 3. {הפער}
**למה זה מזיז:** {...}
**הפעולה הבאה:** {...}
```

Document voice: like a teammate, not like an AI. Specific. "5 of 8 context files are empty, and `Context/icp.md` is all template. Without an ICP, `/morning-report` does not know who to talk about." Not "the context needs completing."

## Step 5: Log the run

Add a log line to `System/logs/YYYY-MM-DD.md` (create the file if there is none) and update the `audit` row in the routines table in `System/health/status.md` (last timestamp + status). Note: the `status.md` skeleton (the one `/connect` and `/doctor` create) has only a `morning-report` row. If there is no `audit` row, add it to the routines table (in the same format: `| audit | {date} | ✅ הצליח | על פי דרישה |`) and then update it. Do not overwrite the `morning-report` row.

```bash
DATE=$(date +%F); TIME=$(date +%H:%M)
mkdir -p System/logs System/health
printf -- '- %s | audit | ✅ הצליח | ציון כולל %s/100, %s פערים דורגו | משך: %ss\n' \
  "$TIME" "$SCORE" "3" "$DUR" >> "System/logs/$DATE.md"
```

(Replace `$SCORE` with the overall score and `$DUR` with the duration in seconds. If the system is shaky, use ⚠️ חלקי instead of ✅ הצליח.)

## Step 6: Summary and recommendation

In the chat, return a short reply: the overall score, one line per gap, and the path to the report. Do not paste the whole report into the chat.

Always end with a recommendation: **"To dig deeper into what to build now, run `/level-up`."** The audit surfaces the gaps, `/level-up` turns them into the next skill.

## Rules

1. Evidence before score. Do not guess how much is built, read `Context/`, `Skills/`, `System/health/status.md` and the logs.
2. A connection counts only with evidence (a connected status or a successful run), not intent.
3. The deciding reason is one specific sentence, not a generic description.
4. Always exactly three gaps, ranked by leverage, each with one concrete action.
5. Markdown only. No HTML. No dashboard.
6. The report is written to `System/health/audit-YYYY-MM-DD.md` with `type: health` frontmatter. Never to the root.
7. Every run writes a log line and updates `System/health/status.md`.
8. Do not paste the full report into the chat. Only a summary and the path.
9. Never use an em dash. Comma, period, colon, or split into sentences.
10. Always end with the recommendation to run `/level-up`.

## Troubleshooting

- **A missing folder (`Context/` or `Skills/`)**: not an error. A low score for that C, and it is probably the top-ranked gap. If the whole structure is missing, recommend running `/onboard` first.
- **No `System/health/status.md`**: a sign that no routine has run. Give a low cadence score, and note that at least one routine needs to be run and scheduled.
- **Context files full of brackets `[...]`**: that is an empty template, not content. Do not count them as filled.
