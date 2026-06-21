---
name: install
description: "Installs a vertical AIOS plugin that the client received as a file (a zip or folder Adir shared via Drive or email) and dropped in their Downloads folder. Finds the package in ~/Downloads, validates it, registers it as a local marketplace, installs the plugin into this AIOS, logs it, and tells the client to restart Claude so the new commands load. Works on macOS, Windows (Git Bash), and Linux. Use when the user says 'התקן', 'התקן תוסף', 'הוסף יכולת', 'התקנה', 'הורדתי תוסף', 'install', 'install plugin', 'add module', 'downloads', or runs /install."
---

# Install a vertical from Downloads

> [!note] התקנת מודול חדש שקיבלת מאדיר. אם אדיר שלח לך קובץ תוסף (zip או תיקייה), שמור אותו בתיקיית ההורדות (Downloads) והפעל את הפקודה. המערכת תתקין את היכולת החדשה לבד.

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, the confirmation, the run-log line, status rows. Keep commands, file paths, plugin names, versions, and dates as they are.

> [!important] OS-agnostic: this skill runs on macOS, Windows (inside Git Bash), and Linux. Detect the OS in step 1 and never assume macOS. Do not rely on tools that are not guaranteed on a fresh client (no `unzip`, no `python3`, no `jq`). Use the portable extraction cascade in step 2, and read JSON manifests with the Read tool, not a shell parser. See the repo `CLAUDE.md` for the full cross-platform rule.

A vertical plugin (for example a real-estate deal-sourcing layer) is a private add-on installed on top of `aios-core`. It is not on a public marketplace, so Adir ships it as a file. The client saves that file in `~/Downloads`, runs `/install`, and this skill installs it locally without the client touching the terminal.

The shape Adir ships is a **marketplace folder**: a directory with `.claude-plugin/marketplace.json` at its root and the plugin under `plugins/<name>/`. The client may have it as a `.zip`, or already unzipped to a folder (macOS and Windows usually auto-extract on download, so the folder case is the common one). This skill handles both.

The flow: step 1 finds the package, step 2 stages it to a stable home, step 3 validates it, step 4 installs it, step 5 logs and reports.

## Step 1: Detect the OS and find the package in Downloads

Detect the system first (the same block `/connect` uses), then look in `~/Downloads`. In Git Bash on Windows, `$HOME/Downloads` resolves to the user's Downloads folder.

```bash
case "$(uname -s)" in
  Darwin) OS=mac ;;
  Linux) OS=linux ;;
  MINGW*|MSYS*|CYGWIN*) OS=windows ;;
  *) OS=unknown ;;
esac
echo "OS: $OS"

DL="$HOME/Downloads"
[ -d "$DL" ] || DL="${USERPROFILE:-$HOME}/Downloads"   # Windows OneDrive-redirected profiles

echo "--- unzipped marketplace folders ---"
find "$DL" -maxdepth 4 -type f -name marketplace.json -path '*/.claude-plugin/*' 2>/dev/null

echo "--- candidate zips ---"
find "$DL" -maxdepth 1 -type f -name '*.zip' 2>/dev/null
```

Decide what to install:
- **One unzipped marketplace folder found**: its root is the parent of `.claude-plugin/`. Use it as `SRC`. Skip extraction in step 2.
- **No folder, but zips found**: the package is still zipped. Pick the zip whose name matches an AIOS vertical (e.g. starts with `aios-`). If it is ambiguous, do not guess: list the zips in Hebrew and ask the client which one Adir sent.
- **Nothing found**: stop. Tell the client in Hebrew that no plugin file was found in Downloads, and ask them to save the file Adir sent into the הורדות folder and run `/install` again. Log nothing as installed.

If more than one valid package is found, list them in Hebrew and ask which one. Never install a package the client did not point at.

## Step 2: Stage to a stable home

The installed marketplace is referenced by its path, so it must live somewhere permanent, not in Downloads (the client may empty that folder). Stage it under `~/.aios/verticals/<name>/`.

```bash
STAGE="$HOME/.aios/verticals"
mkdir -p "$STAGE"
```

**If the client already has a folder** (the common case, auto-extracted on download), `SRC` is that marketplace root directly. Skip the extraction below.

**If it is a zip**, extract it to a temp dir with this portable cascade. It tries whatever the machine actually has, so it works on macOS, Windows (Git Bash), and Linux without assuming any one tool:

```bash
ZIP="<the chosen zip>"
TMP="$(mktemp -d 2>/dev/null || echo "${TMPDIR:-/tmp}/aios-stage-$$")"
mkdir -p "$TMP"

extract_zip() { # $1=zip  $2=destdir
  if command -v unzip   >/dev/null 2>&1; then unzip -oq "$1" -d "$2"; return; fi              # Linux, many macs
  if command -v ditto   >/dev/null 2>&1; then ditto -x -k "$1" "$2"; return; fi               # macOS (always present)
  if command -v 7z      >/dev/null 2>&1; then 7z x -y -o"$2" "$1" >/dev/null; return; fi       # if 7-Zip installed
  if command -v powershell >/dev/null 2>&1; then                                              # Windows (always present)
    powershell -NoProfile -Command "Expand-Archive -LiteralPath '$(cygpath -w "$1")' -DestinationPath '$(cygpath -w "$2")' -Force"; return
  fi
  tar -xf "$1" -C "$2"                                                                         # Win10+/macOS bsdtar fallback
}
extract_zip "$ZIP" "$TMP"

SRC="$(dirname "$(dirname "$(find "$TMP" -type f -name marketplace.json -path '*/.claude-plugin/*' | head -1)")")"
echo "marketplace root inside zip: $SRC"
```

If extraction fails on every tool (rare), do not guess: tell the client in Hebrew to right-click the file and choose "Extract All" (Windows) or double-click it (Mac) to unzip it themselves, then run `/install` again. The folder path in step 1 will pick it up.

**Read the marketplace name from the manifest with the Read tool** (do not parse JSON in the shell, it is not portable). Open `<SRC>/.claude-plugin/marketplace.json` and take its `name` field. Then copy the package to its stable home:

```bash
NAME="<the name value you read from marketplace.json>"
DEST="$STAGE/$NAME"
rm -rf "$DEST"
cp -R "$SRC" "$DEST"
echo "staged to: $DEST"
```

## Step 3: Validate before installing

Never install a package that does not validate. This is the safety gate. `claude plugin validate` is cross-platform.

```bash
claude plugin validate "$DEST" 2>&1
```

If validation fails, stop. Do not register or install anything. Remove the staged copy (`rm -rf "$DEST"`), log `❌ נכשל` with the validation error, and tell the client in Hebrew that the file looks invalid and to ask Adir to resend it.

## Step 4: Register the marketplace and install the plugin

From the same `marketplace.json` you read in step 2, take the marketplace `name` and each entry's `name` under `plugins`. Then add the local marketplace and install each plugin. These `claude` commands are cross-platform and edit the client's plugin config; they do not need a separate terminal.

```bash
# NAME = marketplace name (read from the manifest)
# for each plugin entry name PLUGIN read from the manifest:
claude plugin marketplace add "$DEST" --scope user 2>&1

claude plugin install "<PLUGIN>@$NAME" --scope user 2>&1
# repeat the install line for every plugin the manifest lists
```

Confirm it registered:

```bash
claude plugin list 2>&1
```

If a command fails ("already installed" is fine, a real error is not), log `⚠️` or `❌` with the message and report it to the client in plain Hebrew.

## Step 5: Log, record, and report

List the new commands so the client knows what they gained (the plugin's skill folders are its slash commands):

```bash
ls "$DEST/plugins/<PLUGIN>/skills/" 2>/dev/null
```

Read the plugin's version from `<DEST>/plugins/<PLUGIN>/.claude-plugin/plugin.json` (the `version` field) with the Read tool.

Append a run-log line to `System/logs/YYYY-MM-DD.md` in the standard format:

```
- HH:MM | install | ✅ הצליח | התקנת תוסף <name> v<version> | משך: Ns
```

Add a row to `System/health/status.md` recording the installed vertical: its name, version, install date, and the staged path (so `/doctor` and a future `/update` can find it). Store the path under a `vertical_dir` style field, mirroring `plugin_dir`.

Report to the client in Hebrew, two sentences:
1. What was installed: "התקנתי את התוסף <name> (גרסה X). היכולות החדשות: /command1, /command2, ...".
2. The one action they must take: "סגור ופתח מחדש את Claude כדי שהפקודות החדשות ייטענו".

The restart is required: Claude Code loads plugins at session start, so the new slash commands appear only in the next session.

## Rules

- OS-agnostic always. Detect the OS, never assume macOS, and use only the portable patterns here. No `unzip`/`python3`/`jq` dependency; extract with the cascade, read JSON with the Read tool.
- Install only from the client's own `~/Downloads` (or a folder they explicitly point at). Never fetch from the network here, never install a package the client did not provide.
- The validate gate (step 3) is mandatory. If it fails, install nothing and remove the staged copy.
- If more than one package is found, ask which one. Do not guess.
- Stage to `~/.aios/verticals/<name>/`, not Downloads. The marketplace points at that path, so it must persist.
- Log the version and record the staged path in `status.md`, so the fleet view and updates can track the vertical.
- The client is a consumer: never edit or push the vertical's code from their machine.
