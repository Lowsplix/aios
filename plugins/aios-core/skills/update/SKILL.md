---
name: update
description: "Pulls the latest version of the AIOS plugin from Adir's central repo, so all of the client's skills update without the client doing anything. Runs manually or on a daily schedule for automatic updates. Use when the user says 'update', 'עדכון', 'עדכן את המערכת', or runs /update. Maintainer-facing self-update for the aios-core plugin."
---

# System update

> [!note] Pulls the latest version of the plugin from Adir's repo and updates all the skills. The client does not have to build anything.

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

The skills live in a single Claude Code plugin (`aios-core`) installed from [[אדיר סלם]]'s central repo. When Adir improves a skill at one client, he pushes the change to the repo, and every client pulls it. That keeps everyone up to date from a single point.

## Step 1: Locate the plugin folder

The repo folder is set at install time and stored in `System/health/status.md` under the `plugin_dir` field. If it is not there, look for the cloned `aios` repo.

```bash
PLUGIN_DIR="$(grep -m1 'plugin_dir:' System/health/status.md 2>/dev/null | sed 's/.*plugin_dir: *//')"
[ -z "$PLUGIN_DIR" ] && PLUGIN_DIR="$(find "$HOME" -maxdepth 4 -type d -name aios -path '*aios*' 2>/dev/null | head -1)"
echo "plugin folder: $PLUGIN_DIR"
OLD_VER="$(python3 -c "import json;print(json.load(open('$PLUGIN_DIR/plugins/aios-core/.claude-plugin/plugin.json'))['version'])" 2>/dev/null)"
```

## Step 2: Pull the update

```bash
git -C "$PLUGIN_DIR" pull --ff-only
NEW_VER="$(python3 -c "import json;print(json.load(open('$PLUGIN_DIR/plugins/aios-core/.claude-plugin/plugin.json'))['version'])" 2>/dev/null)"
echo "version: $OLD_VER -> $NEW_VER"
```

Refresh the plugin registration in the terminal with `claude plugin update aios-core` (or `claude plugin marketplace update aios`). Skill changes load in the next session.

## Step 3: Log and report

Update `System/health/status.md` (the `aios-core version` row + timestamp) and add a log line to `System/logs/YYYY-MM-DD.md` in the standard format:

```
- HH:MM | update | ✅ הצליח | עודכן מ-OLD ל-NEW | משך: Ns
```

Report to the client in one Hebrew sentence: if there was no update, "המערכת כבר מעודכנת (גרסה X)". If there was, "עודכנת לגרסה Y, השינויים ייכנסו לתוקף בסשן הבא".

## Scheduling

For automatic operation, schedule `/update` once a day (e.g. 06:00). A cross-platform path: the Claude Desktop app, "Routines" (a local task), or the `schedule` skill. That way the client updates on their own after Adir pushes an improvement. The task runs on this machine, so it needs to be on at the scheduled time. (On macOS you can use `cron` instead. On Windows there is no `cron`.)

## Rules

- Pull only (`--ff-only`). Never change or push code from the client side. The client is a consumer, not an editor.
- If the pull fails (conflict, no network), do not touch anything, log `❌ נכשל` with the reason, and report to Adir.
- Always log the version before and after, so you can tell who is on which version.
