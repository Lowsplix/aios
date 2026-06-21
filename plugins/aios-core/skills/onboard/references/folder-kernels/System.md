# System (the observability: what runs and what broke)

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

This is the health and transparency folder of the system. Every automated routine and every skill action leaves a trace here, so you can always know what ran, when, and whether it succeeded. This is what lets you (and Adir, the maintainer) see the state of the system without guessing. Do not delete files here, they are the operational history. Whatever is rendered here for you is rendered in Hebrew.

## The structure

| Path | What it holds |
| --- | --- |
| `logs/YYYY-MM-DD.md` | Run log for that day. Every routine and skill adds one line here per run |
| `health/status.md` | The single panel: a table of every connection and every routine, when they last ran and what the status is |
| `health/maintainer.md` | The health-ping target (Adir's email). Created the first time `/health-ping` runs |
| `health/audit-YYYY-MM-DD.md` | 4C audit reports, written by `/audit` |

## Log line format (every routine uses this)

Every run adds a line to `logs/YYYY-MM-DD.md` in this format:

```
- HH:MM | skill-name | ✅ הצליח / ⚠️ חלקי / ❌ נכשל | one line: what ran, how much, or the error | משך: Ns
```

Then it updates its row in `health/status.md`: last timestamp, last status, and when the next run is scheduled.

## Who reads from here

- `/doctor` reads `health/status.md` and the latest logs, then probes each component live. This is the panel you open when you ask "is everything working?".
- `/health-ping` sends Adir one health line per day by email, so he knows from a distance what broke. Health summary only, no business content whatsoever.
- `/audit` writes the 4C report here and updates the `audit` row.

## Rules

- Every automated or scheduled run **must** write a log line and update `health/status.md`. Without it, observability is blind.
- Do not delete logs. They are excluded from the backup (`.gitignore` includes `System/logs/`), but stay locally as history.
- `health/maintainer.md` holds the health-ping target only. You can delete the line to disable the ping.
- A log line is an operational fact, not business content. Do not write leads, customer details, or sensitive information here.
