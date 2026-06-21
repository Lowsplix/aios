---
name: doctor
description: "The health and observability surface of the AIOS, for both the client and Adir. Reads System/health/status.md and the latest System/logs, then actively probes each component live: gws auth (auth status + a real Drive list call), wacli (wacli doctor), and the last-run plus last-status of each scheduled routine like morning-report. Prints one Hebrew status table with ✅ תקין / ⚠️ שים לב / ❌ תקול per component, the deciding detail, and exactly what to do to fix anything broken. Then refreshes System/health/status.md with the live findings and appends a run-log line. Use when the user says 'בדיקה', 'דוקטור', 'מה מצב המערכת', 'בדוק את המערכת', 'הכל עובד?', 'doctor', 'health check', 'system status', or runs /doctor."
---

# Doctor (system health check)

> [!note] A live snapshot of the whole system: what is connected, what is running, what is stuck. It checks every component for real (not just reading what is written), shows one Hebrew table with OK / heads-up / broken per component, and for every problem says exactly what to do. At the end it refreshes `System/health/status.md`.

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Run the checks in English, but write everything the user sees in Hebrew: the status table, the summary line, the run-log. Status tokens stay as ✅ תקין / ⚠️ שים לב / ❌ תקול. Keep commands, file paths, and field names as they are.

This is the system dashboard, for you and for Adir. Where `/audit` asks "how much of the system is built", `/doctor` asks "is what is built working right now". It does not trust what is written: it checks live. First it reads what is documented in `System/health/status.md` and the logs, then it actually runs a check on each component and compares.

The structure: step 1 reads the documentation, step 2 checks each component for real, step 3 shows one table, step 4 refreshes the status.

## Step 1: Read the documentation

Read what the system currently thinks about itself. Do not read whole files if you can scan.

```bash
echo "===== status.md ====="
cat System/health/status.md 2>/dev/null || echo "No status.md yet. /connect or /onboard probably has not run."

echo "===== latest log ====="
LAST_LOG=$(ls -t System/logs/*.md 2>/dev/null | head -1)
if [ -n "$LAST_LOG" ]; then
  echo "file: $LAST_LOG"
  tail -15 "$LAST_LOG"
else
  echo "No logs yet. No routine has run."
fi
```

From this extract: which connections are supposed to be active, and when each routine last ran and what its status was. That is the reference point. Now check against reality.

If there is no `status.md` and no logs at all, that is a finding in itself: the system has not been connected yet. Skip to the table (step 3) with everything `❌ תקול`, and one recommendation: run `/connect`.

## Step 2: Live check of each component

Now check for real. Each check is a real command, and the result sets the status. Do not rely on `status.md`, it is only what was. The command is what is.

### 2.1: Google (gws): Gmail, Drive, Calendar

Identity first, then a real read.

```bash
echo "--- gws auth status ---"
gws auth status 2>&1 | head -10

echo "--- live Drive check ---"
gws drive files list --params '{"pageSize": 1}' --format json 2>&1 | head -10
```

Decision:
- Auth active **and** the Drive read came back without error: **✅ תקין**.
- Auth active but the Drive read failed on a disabled service (`SERVICE_DISABLED` / `API not enabled`): **⚠️ שים לב**. Identity is fine, but a specific service is not enabled. Deciding detail: which service. Fix: enable it (see `/connect` step 1.4).
- `gws auth status` shows not connected, or the read returned a permission error (401/403/`invalid_grant`): **❌ תקול**. Needs re-auth. Fix: `/connect` step 1 (or `gws auth login` if the token just expired).

Note: Gmail, Drive, and Calendar share the same gws identity. If the identity is broken, all three are broken. If the identity is fine but only one service is not enabled, mark only that one as `⚠️`.

### 2.2: WhatsApp (wacli)

```bash
echo "--- wacli doctor ---"
wacli doctor 2>&1 | head -20
```

Decision:
- Store healthy, auth active, and search works: **✅ תקין**.
- Connected but with a warning (e.g. sync lagging, or search not indexed): **⚠️ שים לב**. Deciding detail: what doctor flagged. Fix is usually: wait for sync or run `wacli auth --follow` for a moment.
- Not authenticated / store broken / `wacli doctor` failed: **❌ תקול**. Fix: `/connect` step 2 (re-connect with QR).

> [!tip] The `connected` field of `wacli doctor` is not always reliable while sync holds the lock. If it shows not-connected but auth is active and the store is progressing, that is probably just the lock. Mark `⚠️` not `❌`, and note it is worth checking again in a minute.

### 2.3: Data source (API key), if configured

Check only if `.env` has a data-source key (e.g. `GOVMAP_API_KEY`). If not, skip, it is not a fault.

```bash
if [ -f .env ] && grep -q '^GOVMAP_API_KEY=' .env 2>/dev/null; then
  set -a; . ./.env; set +a
  if [ -z "$GOVMAP_API_KEY" ] || [ "$GOVMAP_API_KEY" = "PASTE_KEY_HERE" ]; then
    echo "Data-source key is set but empty or not pasted. BROKEN"
  else
    echo "Data-source key loaded (${#GOVMAP_API_KEY} chars). OK, will probe the service if there is a health endpoint."
    # If a service health endpoint is known, run it here and decide by the response code:
    # curl -s -o /dev/null -w '%{http_code}\n' -H "Authorization: Bearer $GOVMAP_API_KEY" "https://API_BASE/health"
  fi
else
  echo "No data source configured. Skipping (not a fault)."
fi
```

Decision: key loaded (and the service probe passed, if any) = **✅ תקין**. Key missing/empty = **❌ תקול**. A service probe that returned 401/403 = **⚠️ שים לב** (key probably expired).

### 2.4: Routines (morning-report and others)

Routines are not checked by running them (do not run them), but by their last run in the logs and status. The question: when did it last run, and what was the status.

```bash
echo "--- recent routine runs (7 days) ---"
ls -t System/logs/*.md 2>/dev/null | head -7 | while read f; do
  grep -E 'morning-report|supervisor' "$f" 2>/dev/null
done | tail -10
echo "--- today ---"; date +%F
```

Decision per routine (e.g. `morning-report`):
- Ran today or yesterday with ✅ success: **✅ תקין**.
- Ran but the last status was ⚠️ partial or ❌ failed: **⚠️ שים לב** or **❌ תקול** accordingly. Deciding detail: the line from the log (what failed). Fix: run it manually once and see what breaks.
- Has not run in more than a week, or has no run in the logs at all: **⚠️ שים לב** (stuck or not scheduled). Fix: confirm it is scheduled and the computer was awake at the time, and run it manually once.

> [!important] Routines run only when the computer and terminal are on at the scheduled time. If a routine "did not run", the most common reason is that the computer was off, not that it is broken. Note this when relevant.

## Step 3: The status table

Show one clear table in the chat, in Hebrew. Per component: status, the deciding detail, and what to do. Three levels: **✅ תקין** (working), **⚠️ שים לב** (working partially or needs attention), **❌ תקול** (not working, needs handling). For a healthy row, the "what to do" column says "כלום, תקין".

Render this table to the user in Hebrew (translate the column headers; keep the status tokens as they are):

```markdown
## מצב המערכת ({date} {time})

| רכיב | מצב | הפרט שהכריע | מה לעשות |
| --- | --- | --- | --- |
| ג'ימייל (gws) | {✅/⚠️/❌} | {detail} | {action or "כלום, תקין"} |
| דרייב (gws) | {✅/⚠️/❌} | {detail} | {...} |
| יומן (gws) | {✅/⚠️/❌} | {detail} | {...} |
| וואטסאפ (wacli) | {✅/⚠️/❌} | {detail} | {...} |
| מקור נתונים (API) | {✅/⚠️/❌/–} | {detail} | {...} |
| morning-report | {✅/⚠️/❌} | {last run + status} | {...} |
```

Below the table, one summary line in human Hebrew. Specific, not generic:
- If all ✅: "הכל עובד. המערכת מחוברת והשגרה רצה. אין מה לעשות."
- If there are problems: one line on the most urgent problem and the next step. For example: "הכל תקין חוץ מוואטסאפ, שהתנתק. הרץ `/connect` ושלב 2 כדי לחבר מחדש בסריקת QR."

A human, reassuring voice, not alarming. Even ❌ is said calmly: "X לא מחובר כרגע, זה תיקון של דקה."

## Step 4: Refresh status.md and log

Update `System/health/status.md` to reflect what was found now. This keeps the dashboard in sync with reality, so the next read (and `/audit`) sees the truth.

Edit `System/health/status.md`: for each connection, update the **state** column from the live check (✅ מחובר / ⚠️ שים לב / ❌ תקול) and **updated** to now. For each routine, update last run and last state. If the file does not exist at all, create it with this skeleton (English structure, Hebrew display values) and then fill:

```bash
mkdir -p System/health System/logs
if [ ! -f System/health/status.md ]; then
cat > System/health/status.md <<'EOF'
---
type: health
status: active
tags: [health, status, connections]
---

> [!note] מצב המערכת: מה מחובר ומה רץ. מתעדכן בכל `/connect` ובכל `/doctor`.

## Connections

| מערכת | מצב | עודכן |
| --- | --- | --- |
| ג'ימייל (gws) | ⬜ לא מחובר | - |
| דרייב (gws) | ⬜ לא מחובר | - |
| יומן (gws) | ⬜ לא מחובר | - |
| וואטסאפ (wacli) | ⬜ לא מחובר | - |
| מקור נתונים (API) | ⬜ לא מחובר | - |

## Routines

| שגרה | ריצה אחרונה | מצב אחרון | תזמון |
| --- | --- | --- | --- |
| morning-report | - | - | - |
EOF
fi
```

Append a log line for this run:

```bash
DATE=$(date +%F); TIME=$(date +%H:%M); mkdir -p System/logs
printf -- '- %s | doctor | %s | %s | משך: %ss\n' \
  "$TIME" "$RESULT" "$SUMMARY" "$DUR" >> "System/logs/$DATE.md"
```

(`$RESULT` = ✅ הצליח if everything is fine, ⚠️ חלקי if there are problems but the check ran, ❌ נכשל if the check itself could not run. `$SUMMARY` = one line, e.g. "הכל תקין" or "וואטסאפ מנותק, gws תקין". `$DUR` = duration in seconds.)

## Rules

1. Check live, do not trust `status.md`. It is only a reference point. The command (gws, wacli) sets the status.
2. Every component gets one of three: ✅ תקין, ⚠️ שים לב, ❌ תקול. Each with the deciding detail and what to do.
3. For a healthy component, "what to do" is "כלום, תקין". Do not invent problems.
4. Routines are measured by their last run in the logs (do not run them). A routine that has not run for a week = `⚠️`, not a certain failure. Remember the computer needs to be on at the time.
5. After the check, refresh `System/health/status.md` to the live state, and append a log line to `System/logs/`.
6. A reassuring, human voice. Even ❌ is said calmly, with the next step. Without scaring.
7. If there is no `status.md` and no logs: the system is not connected. Recommend `/connect`, do not treat it as a fault.
8. Do not paste all the raw check output into the chat. Only the table and the summary.
9. Never write an API key value to the chat or a vault file. Check that it loaded, not what it is.
10. Never use an em dash. Comma, period, colon, or split into sentences.
11. Everything the user reads is in Hebrew: the table, the summary, the recommendation.

## Troubleshooting

- **`gws auth status` is fine but the Drive read fails**: almost always a service not enabled in the project. That is `⚠️` (not `❌`): identity is fine. Point to `/connect` step 1.4 to enable Gmail/Drive/Calendar.
- **`gws` returns `invalid_grant` or demands login**: the token expired or the session was revoked. `❌`. Quick fix: `gws auth login`. If that does not help, `/connect` step 1 from the start.
- **`wacli doctor` is stuck or slow**: sync is probably holding the lock. Do not declare `❌` right away. Mark `⚠️`, note it is worth re-checking in a minute, and if it stays stuck run `wacli auth --follow` for a moment then `wacli doctor` again.
- **A routine with no run in the logs**: it may never have been scheduled, or it was scheduled but the computer was off at the time. `⚠️`. Confirm it is scheduled, run it manually once, and see that a log line is written.
- **All checks fail together**: the `gws`/`wacli` commands may not be installed at all, or you are not in the vault directory. Confirm you are at the vault root (it has `System/` and `Context/`), and that Adir installed the tools. That is a check failure (`❌ נכשל` in the log), not a single-component fault.
