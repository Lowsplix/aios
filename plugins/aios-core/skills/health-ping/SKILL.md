---
name: health-ping
description: "Runs a health check and sends one status line to Adir (the maintainer) by email through gws, so he has a daily pulse on every client system without logging into each machine. Cross-platform (works on Windows too). Sends only a health summary, no business content. Runs on a daily schedule. Use when the user says 'health ping', 'פינג בריאות', 'שלח סטטוס לאדיר', or runs /health-ping. Maintainer fleet-monitoring heartbeat over email."
---

# Health ping to Adir

> [!note] Sends Adir one email a day with a single health line, so he knows from a distance what is working and what broke at each client. Only a health summary, no private business content.

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

This is Adir's observability layer across all clients. Every client system sends a daily pulse **by email through [[gws]]**, a channel that works on every OS (including Windows, where there is no wacli). Adir filters these emails into one label and sees the whole fleet in a single panel.

## Step 1: Health check

Run the logic of `/doctor` (or read `System/health/status.md` if it just ran): the state of the connections (gws, wacli if relevant, API keys) and the last run of each routine. Summarize into a single line, for example:

`STATUS_LINE="[client name] | 2026-06-18 | gws ✅ | דוח בוקר ✅ 07:00 | סריקה ❌ אתמול | גרסה 0.1.0"`

## Step 2: Maintainer target (Adir)

The target is stored in `System/health/maintainer.md`. If the file does not exist, create it with the default (Adir), so the pulse is wired up from the very first moment:

```bash
MF="System/health/maintainer.md"
if [ ! -f "$MF" ]; then
  mkdir -p System/health
  cat > "$MF" <<'EOF'
---
type: config
---

# יעד המתחזק (מי מקבל את דופק הבריאות)

אדיר סלם הגדיר את עצמו כמתחזק המערכת. הדופק היומי נשלח אליו במייל, ומכיל **רק** סיכום בריאות (אילו רכיבים תקינים, מה נשבר), בלי שום תוכן עסקי. אפשר לבטל בכל רגע על ידי מחיקת השורה.

email: adir@adirsellam.com
EOF
fi
EMAIL="$(grep -m1 '^email:' "$MF" | sed 's/^email: *//')"
echo "target: $EMAIL"
```

## Step 3: Send by email through gws

The default channel. Works on every OS (gws is a Node tool):

```bash
if command -v gws >/dev/null 2>&1 && gws auth status >/dev/null 2>&1 && [ -n "$EMAIL" ]; then
  CLIENT="$(grep -m1 '^email:' Context/me.md 2>/dev/null | sed 's/^email: *//')"
  gws gmail +send --to "$EMAIL" \
    --subject "AIOS health: ${CLIENT:-client} $(date +%F)" \
    --body "$STATUS_LINE" && SENT="✅ הצליח"
else
  SENT="⚠️ חלקי"
  echo "$STATUS_LINE" >> System/health/status.md
  echo "gws not connected or no target, the pulse was saved locally to System/health/status.md"
fi
```

> [!note] If the client is on a Mac and has wacli connected, you can additionally send over WhatsApp as a bonus, but email is the primary channel because it is cross-platform.

## Step 4: Log

Add a log line to `System/logs/YYYY-MM-DD.md`:

```
- HH:MM | health-ping | ✅ הצליח | נשלח דופק לאדיר במייל | משך: Ns
- HH:MM | health-ping | ⚠️ חלקי | gws לא מחובר, דופק נשמר מקומית | משך: Ns
```

## Scheduling

Schedule it once a day (e.g. 07:05, right after the morning report). A cross-platform path: the Claude Desktop app, "Routines" (a local task), or the `schedule` skill. The task runs on this machine, so it must be on at the scheduled time. (On macOS you can use `cron` instead. On Windows there is no `cron`.)

## Rules

- Send only a health summary: component names, ✅/⚠️/❌, timestamps, version. Never business content, no leads, no client details.
- The channel is email through gws (cross-platform). That is the default and works on Windows too.
- The target is defined in `System/health/maintainer.md`. Default: adir@adirsellam.com. The client can delete it to opt out.
- If gws is not connected, do not fail: save the pulse locally and log ⚠️ חלקי.
