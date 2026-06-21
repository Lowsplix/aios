---
name: morning-report
description: "Composes a concise Hebrew morning brief for a solo business owner by reading the latest Daily/ note, open tasks in task-list/Tasks.md, flagged items in active Projects/, leads, and System/health/status.md. Produces five Hebrew sections (new opportunities, decisions to make, today's tasks, leads, one system-health line), writes the brief to today's Daily/YYYY-MM-DD.md, optionally sends it to the owner over WhatsApp via wacli (after a wacli doctor liveness check), appends a run-log line to System/logs/YYYY-MM-DD.md and updates System/health/status.md. Designed to be scheduled around 07:00 (machine must be on). Use when the user says 'דוח בוקר', 'בוקר', 'מה יש לי היום', 'סיכום בוקר', 'morning report', 'morning brief', or runs /morning-report."
---

# Morning report

> [!note] Composes a short Hebrew morning brief from what already lives in the vault: the latest daily note, open tasks, what is flagged in projects, new leads, and one health line. It writes it to today's note, and if you want, also sends it to you over WhatsApp.

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

The report invents nothing. It reads what is already written in the vault and serves it to you filtered down to the 5 things that matter this morning: new opportunities, decisions waiting for you, what to do today, leads, and system health in one line. The goal: that you open the day with one clear picture, without searching.

This is steps. Run them in order. Speak Hebrew, short and factual. Specific, not generic.

## Step 1: Gather the evidence

Gather what you need to compose the brief. Do not read whole files if you can scan. Use `ls`, `grep`, and `Read` only on the relevant file.

```bash
DATE=$(date +%F)

# the latest daily note (the current context)
ls -t Daily/*.md 2>/dev/null | head -1

# open tasks
grep -nE '^\s*-\s*\[ \]' task-list/Tasks.md 2>/dev/null

# what is flagged in active projects: deadline, blocked, awaiting a decision
grep -rniE 'דדליין|חסום|תקוע|ממתין|להחליט|החלטה|בלוקר|blocker' Projects/ 2>/dev/null

# leads: files or records marked as a new lead
grep -rniE 'ליד|leads?|פנייה חדשה|לקוח פוטנציאלי' Projects/ Intelligence/ 2>/dev/null | head -20

# system health: one line
cat System/health/status.md 2>/dev/null
```

What to extract from each source:
- **The latest daily note in `Daily/`**: what happened yesterday, what is still open, what is set for today. This is the context the morning is built from.
- **`task-list/Tasks.md`**: all open tasks (`- [ ]`). Flag the ones due today or overdue, and the high-priority ones.
- **`Projects/`**: every active project flagged with a near deadline, a blocker, or something awaiting your decision. These feed "decisions to make".
- **Leads**: new inquiries or potential customers that came in and have not been handled yet. If there is no organized lead source in the vault, keep the section short or write "no new leads".
- **`System/health/status.md`**: the last line on system state. What ran recently, what failed, what is connected. From this the one health line is built.

If a folder or file is missing, that is not an error. Skip that section or write that there is no info, and continue.

## Step 2: Compose the brief

From the evidence, compose a short Hebrew brief with the five sections. Read `references/report-template.md` to follow the structure and headings exactly.

Writing rules:
- Every section short. Bullets, not paragraphs. If a section has no content, write one line in it ("no new opportunities this morning") and do not delete the heading.
- Specific, not generic. "Decide whether to take the [[name]] project, it is blocking the week's schedule" and not "there are open decisions".
- `[[wikilinks]]` for every entity: person, company, project, lead. Weave into the sentence.
- **System health in one line only.** For example: "Everything running. Morning report yesterday 07:00 ✅, WhatsApp connected." or: "⚠️ The WhatsApp sync has not run since yesterday, worth checking."

The five sections (in this order):

| Section | What goes in |
| --- | --- |
| New opportunities | New things worth jumping on: a hot lead, an inquiry, an opening that came up in the daily note or a project. |
| Decisions to make | What is waiting for your call and blocking progress. From projects and the daily note. |
| Today's tasks | From the open tasks: what is urgent, what is due today, the 1 to 3 most important. |
| Leads | New inquiries not handled yet, with the source if known. |
| System health | One line: what ran, what failed, what is connected. |

## Step 3: Write the report to the vault

Write the brief to today's note `Daily/YYYY-MM-DD.md` (today's date). If the file already exists, **merge** the brief into it under a `## דוח בוקר` heading instead of overwriting. If there is no daily note for today yet, create it with the standard frontmatter.

Frontmatter structure:

```yaml
---
type: report
date: YYYY-MM-DD
status: active
tags: [morning-report, daily]
---
```

If you prefer a separate brief from the daily note, you can write it to `Reports/morning-YYYY-MM-DD.md` (create `Reports/` if it does not exist). The default is the daily note, because that is where the morning gets looked for.

## Step 4: Send over WhatsApp (optional)

If the user asked for the report to be sent to them over WhatsApp (and that is the default for the scheduled routine), send it via `wacli` to the owner.

> [!warning] Liveness check first. `wacli` works on every operating system (Mac, Windows, Linux), but it may not be installed and connected yet. If it is not installed, that is fine, do not fail over it. The brief was already written to the daily note in step 3, and that is the important result. If `wacli` is not available, skip the send, log `⚠️ חלקי` with the line "WhatsApp לא זמין: wacli לא מותקן", and note in the system-health line that the brief was written but not sent. To connect WhatsApp, run `/connect` and choose WhatsApp.

Check first whether `wacli` is installed and working. If the command is not found, the tool has not been installed and connected yet: skip all of step 4 and continue to step 5 with a partial status.

```bash
# availability gate + liveness gate: without both, do not send
if ! command -v wacli >/dev/null 2>&1; then
  echo "wacli is not installed yet. Brief written to the vault, not sent over WhatsApp."
else
  wacli doctor
fi
```

If `wacli` exists but `wacli doctor` shows the sync is stalled or disconnected: do not send. Write the report to the vault anyway, and note in the system-health line that the send was skipped because WhatsApp was not connected.

If everything is fine, send the brief to the owner (the target number is saved in `Context/me.md` or `Context/infrastructure.md`):

```bash
# replace OWNER with the owner's number/id from the context
wacli send --to "$OWNER" --text "$BRIEF"
```

The WhatsApp brief should be a short, clean version: a short title, then the five sections in simple lines without Obsidian syntax (no `[[ ]]`, no `> [!note]`). WhatsApp shows plain text.

## Step 5: Run-log and health

Every run writes a log line and updates the system state, per the AIOS convention.

```bash
DATE=$(date +%F); TIME=$(date +%H:%M); mkdir -p System/logs System/health
printf -- '- %s | morning-report | ✅ הצליח | בריף נכתב להערת היום%s | משך: %ss\n' \
  "$TIME" "$SENT_NOTE" "$DUR" >> "System/logs/$DATE.md"
```

(`$SENT_NOTE` = ", נשלח בוואטסאפ" if sent, otherwise empty. `$DUR` = duration in seconds. If the send was skipped because WhatsApp was not connected, use ⚠️ חלקי instead of ✅ הצליח, and write in the line that the brief was written but not sent.)

Update the `morning-report` line in `System/health/status.md`: last timestamp, last status, and the next schedule. If there is no table yet, create it with columns: routine, last run, status, next schedule.

## Step 6: Summary in the chat

Return a short reply: an opening line ("the brief is ready"), then the headings with one line per section, and the path to the file written. If it was sent over WhatsApp, note that. Do not paste the whole report twice.

## Scheduling

The report is built to run on its own every morning, around 07:00, before you start the day. It does not run by itself: it needs a trigger wired up.

> [!important] The computer must be on and open at run time. The system is a push of a result during your working hours, not a server that runs 24/7. If the computer is off at 07:00, the report simply will not run that day.

Two ways to schedule, both work on Windows and macOS:

1. **The Claude Desktop app, "Routines" (a local task)**: in the Claude Desktop app you can set up a local task that runs `/morning-report` every day at 07:00. The task runs on this computer, so it must be on at that time. This is the simplest way, and it is cross-platform.

2. **The schedule skill** (`schedule`): run the `schedule` skill and ask it to schedule `/morning-report` every day at 07:00.

(On macOS only, if preferred: you can use a `cron` line instead. Windows has no `cron`, use the two ways above.)

After scheduling, update the `morning-report` line in `System/health/status.md` with "next schedule: tomorrow 07:00", so `/doctor` knows when the report is supposed to run.

## Rules

1. Do not invent. The brief is built only from what is already written in the vault. If there is no info for a section, write that there is none, do not fill it in out of thin air.
2. Exactly five sections, in this order: new opportunities, decisions to make, today's tasks, leads, system health.
3. System health is **one line only**.
4. Specific, not generic. With `[[wikilinks]]` for every entity in the vault file.
5. The report is written to today's note `Daily/YYYY-MM-DD.md` (or `Reports/`), with `type: report` frontmatter. Never to the vault root.
6. Send over WhatsApp only after `wacli doctor`. If WhatsApp is not connected, do not send, but do write to the vault and note that the send was skipped.
7. Every run writes a log line to `System/logs/` and updates `System/health/status.md`.
8. The WhatsApp version is clean text without Obsidian syntax.
9. Never use an em dash. Comma, period, colon, or split into sentences.
10. Do not paste the whole report in the chat. A short summary and a path.

## Troubleshooting

- **`wacli doctor` shows the sync is stalled**: do not send, the send would go into the void. Write the brief to the vault, mark ⚠️ חלקי in the log, and note in the health line that the WhatsApp connection is worth checking.
- **No previous daily note in `Daily/`**: not an error. Build the brief from the tasks and projects only, and note that "there is no context from yesterday".
- **`task-list/Tasks.md` is missing**: setup was probably not completed. Build a partial brief from what is available, and note in the health line that it is worth running `/onboard` or checking the vault structure.
- **No lead source in the vault**: keep the leads section short ("no new leads"). That is fine, not every vault manages leads.
