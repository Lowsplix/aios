# wacli always-on supervision (maintainer reference)

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

This is the battle-tested recipe for keeping `wacli sync` connected to WhatsApp continuously and surviving every crash class we have actually hit in production. It is a maintainer/operator reference (English, technical), not client-facing. The `/connect` skill installs the OS-appropriate version of this after `wacli auth`; the `/supervisor` skill relies on the liveness gate below.

WhatsApp multi-device keeps a persistent WebSocket from the laptop as a peer (the phone does not need to be online). The job of this stack is: that WebSocket is always up, and when it silently dies, it comes back without a human.

## The model: three layers

| Layer | Job | Catches |
| --- | --- | --- |
| 1. Sync, kept alive | run `wacli sync --follow --download-media`, auto-restart on exit/network change | process crash, clean exit, network bounce |
| 2. Connection-aware watchdog | every ~2 min, detect an alive-but-disconnected sync and force a clean restart | the zombie: process alive, WebSocket dead, stuck reconnecting. KeepAlive structurally cannot see this |
| 3. Transcription (optional) | every ~10 min, whisper.cpp turns voice notes into sidecar `.txt` | `[Audio]` placeholders the digest cannot read |

Layer 1 alone is NOT enough, and that is the whole point. The failure that motivated layer 2 is below.

## Two hard-won lessons (do not relearn these)

1. **`wacli doctor`'s `connected` field is a lie while sync is running.** When `wacli sync --follow` owns the store lock (always, under this setup), `wacli --read-only --json doctor` returns `connected: false` and `connection_state: locked_by_other_process`. That is doctor's own failed connect attempt, NOT the sync process's real WebSocket state. **Authoritative liveness signals, in order:** (a) the last connection token in the sync error log (`Connected.` vs `Disconnected.`/`Reconnecting`), (b) `data.store.last_sync_at` from doctor, (c) whether the newest message timestamp keeps advancing. Never gate on `connected`.

2. **KeepAlive cannot catch the zombie.** A `sync --follow` process can stay alive for days while its WebSocket is dead, spinning in `Disconnected -> Reconnecting` (often 403/429 rate-limited after sleep/wake). The process never exits, so KeepAlive never fires. Only a fresh process restart re-establishes a clean socket, and you have to `kill` the wedged PID first because it holds the store lock (a plain `launchctl kickstart` cannot displace it).

## The failure-mode case study (2026-05-26)

`wacli sync --follow` (PID 13139) alive since May 25 09:54, but stuck in a silent reconnect loop since May 25 11:35. Store had not advanced since `2026-05-24T12:20:15Z`. KeepAlive missed it (process never exited). The operator's daily digest correctly read the store and reported "nothing happened" across four work-tagged contacts, who had in fact messaged. Root cause: a prior session was told "keep sync running" and only verified the process existed, not that it was connected. Fix: `kill` the zombie, `kickstart` a fresh process, auth was still valid, backfill caught up. The watchdog below now catches this class within ~2 to 4 minutes.

## The liveness gate (every digest routine must do this)

Any routine that produces a "what happened on WhatsApp" output (the supervisor, a morning digest) must gate first, or an empty store and a dead sync look identical:

```bash
LS=$(wacli --read-only --json doctor 2>/dev/null \
  | python3 -c 'import sys,json;print(json.load(sys.stdin).get("data",{}).get("store",{}).get("last_sync_at",""))')
# if LS is older than the digest window, emit "sync stalled, last sync at $LS" and STOP.
# Do NOT emit a "nothing happened" digest. Never trust doctor.data.connected.
```

Data-source split for the digest itself: tags / contact metadata / arbitrary filters the CLI does not expose come from **SQL on `~/.wacli/wacli.db`** (`contact_tags`, `contacts`); message bodies, search, context, and sending come from **`wacli --read-only` subcommands** (they handle WAL + FTS5 and coexist with the sync lock).

---

## macOS implementation (proven, verbatim)

Three LaunchAgents under `~/Library/LaunchAgents/`, logs under `~/Library/Logs/wacli/`. This is the exact production setup on Adir's Mac.

### Layer 1: `sh.wacli.sync.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key><string>sh.wacli.sync</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/wacli</string>
        <string>sync</string>
        <string>--follow</string>
        <string>--download-media</string>
    </array>
    <key>RunAtLoad</key><true/>
    <key>KeepAlive</key>
    <dict>
        <key>SuccessfulExit</key><false/>
        <key>NetworkState</key><true/>
    </dict>
    <key>ThrottleInterval</key><integer>30</integer>
    <key>ProcessType</key><string>Background</string>
    <key>StandardOutPath</key><string>/Users/adir/Library/Logs/wacli/sync.out.log</string>
    <key>StandardErrorPath</key><string>/Users/adir/Library/Logs/wacli/sync.err.log</string>
    <key>EnvironmentVariables</key>
    <dict><key>PATH</key><string>/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin</string></dict>
</dict>
</plist>
```

`KeepAlive { SuccessfulExit=false, NetworkState=true }` restarts on crash and on network change. `ThrottleInterval=30` prevents a tight crash loop.

### Layer 2: `sh.wacli.watchdog.plist` (StartInterval 120) + `~/.wacli/watchdog.sh`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key><string>sh.wacli.watchdog</string>
    <key>ProgramArguments</key>
    <array><string>/bin/bash</string><string>/Users/adir/.wacli/watchdog.sh</string></array>
    <key>RunAtLoad</key><true/>
    <key>StartInterval</key><integer>120</integer>
    <key>ProcessType</key><string>Background</string>
    <key>StandardOutPath</key><string>/Users/adir/Library/Logs/wacli/watchdog.stdout.log</string>
    <key>StandardErrorPath</key><string>/Users/adir/Library/Logs/wacli/watchdog.stderr.log</string>
    <key>EnvironmentVariables</key>
    <dict><key>PATH</key><string>/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin</string></dict>
</dict>
</plist>
```

`~/.wacli/watchdog.sh` (v2, the crown jewel: connection-token primary signal, kill-before-kickstart, store-progress backstop, self-rotating log):

```bash
#!/usr/bin/env bash
# wacli connection-aware watchdog (v2).
# Guards the alive-but-disconnected zombie: sync process alive, WebSocket dead,
# spinning Disconnected -> Reconnecting. KeepAlive cannot catch it (no exit).
# PRIMARY signal: last connection token in the sync error log (independent of
# message volume, so no false alarms when genuinely quiet, and not reset by
# sleep/wake). BACKSTOP: store progress stalled, only when the token is unreadable.
set -euo pipefail

WACLI=/opt/homebrew/bin/wacli
SYNC_LABEL=sh.wacli.sync
SYNC_MATCH="wacli sync --follow"
STORE_DIR="$HOME/.wacli"
STATE_FILE="$STORE_DIR/.watchdog-state"
LOG_DIR="$HOME/Library/Logs/wacli"
LOG="$LOG_DIR/watchdog.log"
ERR_LOG="$LOG_DIR/sync.err.log"

STALE_SECONDS=21600   # 6h of zero store progress, only used when token unreadable
DOWN_CONFIRM=2        # consecutive down observations before acting (debounce)
LOG_MAX_BYTES=$((5 * 1024 * 1024))

mkdir -p "$LOG_DIR"
ts()  { date -u +"%Y-%m-%dT%H:%M:%SZ"; }
log() { echo "$(ts) $*" >> "$LOG"; }

read_state() { /usr/bin/python3 -c "
import json
try:
    s=json.load(open('$STATE_FILE')); print(s.get('$1','$2'))
except Exception: print('$2')"; }
write_state() { /usr/bin/python3 -c "
import json
json.dump({'newest':'''$1''','observed_at':$2,'down_count':$3}, open('$STATE_FILE','w'))"; }

kickstart() {
    log "RESTART $SYNC_LABEL: $1"
    /usr/bin/pkill -f "$SYNC_MATCH" 2>/dev/null || true
    /bin/sleep 1
    /usr/bin/pkill -9 -f "$SYNC_MATCH" 2>/dev/null || true   # KILL: it holds the store lock
    /bin/launchctl kickstart -k "gui/$(id -u)/$SYNC_LABEL" || log "kickstart non-zero"
}

# Cap the error log so the reconnect loop cannot fill the disk.
if [ -f "$ERR_LOG" ]; then
    SIZE=$(/usr/bin/stat -f%z "$ERR_LOG" 2>/dev/null || echo 0)
    if [ "$SIZE" -gt "$LOG_MAX_BYTES" ]; then
        /usr/bin/tail -n 300 "$ERR_LOG" > "$ERR_LOG.keep" 2>/dev/null || true
        : > "$ERR_LOG"; /bin/cat "$ERR_LOG.keep" >> "$ERR_LOG" 2>/dev/null || true
        /bin/rm -f "$ERR_LOG.keep"; log "rotated sync.err.log (was ${SIZE} bytes)"
    fi
fi

PREV_NEWEST=$(read_state newest "")
PREV_OBSERVED=$(read_state observed_at 0)
DOWN_COUNT=$(read_state down_count 0)
NOW_EPOCH=$(date -u +%s)

# 1. Process liveness (KeepAlive should cover this; nudge if wedged).
if ! /usr/bin/pgrep -f "$SYNC_MATCH" >/dev/null; then
    kickstart "sync process not found"; write_state "$PREV_NEWEST" "$NOW_EPOCH" 0; exit 0
fi

# 2. Connection state from the sync error log (last connection token).
CONN=unknown
if [ -f "$ERR_LOG" ]; then
    STATE_LINE=$(/usr/bin/grep -aoE "Connected\.|Disconnected\.|Reconnecting" "$ERR_LOG" 2>/dev/null | /usr/bin/tail -n 1 || true)
    case "$STATE_LINE" in
        "Connected.")                   CONN=up ;;
        "Disconnected."|"Reconnecting") CONN=down ;;
        *)                              CONN=unknown ;;
    esac
fi

if [ "$CONN" = "down" ]; then
    DOWN_COUNT=$(( DOWN_COUNT + 1 ))
    if [ "$DOWN_COUNT" -ge "$DOWN_CONFIRM" ]; then
        kickstart "connection down: reconnect loop (${DOWN_COUNT} checks)"
        write_state "$PREV_NEWEST" "$NOW_EPOCH" 0; exit 0
    fi
    log "connection down (check ${DOWN_COUNT}/${DOWN_CONFIRM}), watching"
    write_state "$PREV_NEWEST" "$PREV_OBSERVED" "$DOWN_COUNT"; exit 0
fi
DOWN_COUNT=0   # up or unknown: clear the counter

# 3. Store-progress backstop (only when the connection token is unreadable).
NEWEST=$("$WACLI" --read-only --json messages list --limit 1 2>/dev/null \
    | /usr/bin/python3 -c 'import sys,json
try:
    m=json.load(sys.stdin).get("data",{}).get("messages",[]); print(m[0]["Timestamp"] if m else "")
except Exception: print("")' 2>/dev/null || true)

if [ -z "$NEWEST" ]; then write_state "$PREV_NEWEST" "$PREV_OBSERVED" 0; exit 0; fi
if [ "$NEWEST" != "$PREV_NEWEST" ]; then write_state "$NEWEST" "$NOW_EPOCH" 0; exit 0; fi
if [ "$PREV_OBSERVED" -eq 0 ]; then write_state "$NEWEST" "$NOW_EPOCH" 0; exit 0; fi

STUCK_FOR=$(( NOW_EPOCH - PREV_OBSERVED ))
if [ "$CONN" = "unknown" ] && [ "$STUCK_FOR" -gt "$STALE_SECONDS" ]; then
    kickstart "backstop: no store progress ${STUCK_FOR}s and token unknown (newest=$NEWEST)"
    write_state "$NEWEST" "$NOW_EPOCH" 0; exit 0
fi
write_state "$NEWEST" "$PREV_OBSERVED" 0; exit 0
```

Install (macOS):

```bash
mkdir -p ~/Library/Logs/wacli
# write the two plists and watchdog.sh as above, then:
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/sh.wacli.sync.plist
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/sh.wacli.watchdog.plist
```

### Layer 3 (optional): `sh.wacli.transcribe.plist` (StartInterval 600) + `~/.wacli/transcribe.sh`

Sweeps `~/.wacli/media/**/*.oga`, skips any that already has a sibling `<name>.txt`, decodes Opus to 16kHz mono PCM with ffmpeg, runs `whisper-cli -m ggml-large-v3-turbo.bin -l auto`, writes the transcript as the sidecar `.txt`. mkdir-based lock (`~/.wacli/.transcribe.lock.d`) since macOS has no `flock`. The digest must read the sidecar (`.oga` -> `.txt`) for any `MediaType: audio` message, falling back to `[transcript pending]`, never showing `[Audio]`. Model `ggml-large-v3-turbo.bin` (~1.5GB) from huggingface.co/ggerganov/whisper.cpp, runs on Metal, ~2 to 5s per note.

### macOS ops cheatsheet

```bash
launchctl list | grep wacli                              # expect sync + watchdog (+ transcribe)
launchctl kickstart -k gui/$(id -u)/sh.wacli.sync        # force a clean restart now
tail ~/Library/Logs/wacli/sync.err.log                   # healthy: "Connected." then quiet; bad: repeating Reconnecting
tail ~/Library/Logs/wacli/watchdog.log                   # only writes when it acts; silence = healthy
wacli --read-only --json doctor | python3 -c 'import sys,json;print(json.load(sys.stdin)["data"]["store"]["last_sync_at"])'
# Re-auth (only if WhatsApp truly revokes the link):
launchctl bootout gui/$(id -u)/sh.wacli.sync; wacli auth; launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/sh.wacli.sync.plist
```

---

## Windows implementation (ported, validate on first deploy)

Same three layers, same two lessons, same watchdog logic. Windows has no launchd, so Task Scheduler runs the jobs and PowerShell replaces bash. wacli is the same Go binary and writes the same `Connected.`/`Disconnected.`/`Reconnecting` tokens, so the connection-token signal ports directly. Paths assume `wacli.exe` on `PATH` and the store at `%USERPROFILE%\.wacli`.

### Layer 1: keep sync alive

PowerShell wrapper `%USERPROFILE%\.wacli\sync-loop.ps1` (Task Scheduler has no real KeepAlive, so a relaunch loop with throttle plays that role):

```powershell
$ErrorActionPreference = "SilentlyContinue"
$log = "$env:USERPROFILE\.wacli\logs"; New-Item -ItemType Directory -Force -Path $log | Out-Null
while ($true) {
    & wacli sync --follow --download-media `
        1>> "$log\sync.out.log" 2>> "$log\sync.err.log"
    Start-Sleep -Seconds 30   # throttle, mirrors ThrottleInterval=30
}
```

Register it at logon (runs whenever the machine is on and the user is logged in):

```powershell
$action  = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-WindowStyle Hidden -ExecutionPolicy Bypass -File `"$env:USERPROFILE\.wacli\sync-loop.ps1`""
$trigger = New-ScheduledTaskTrigger -AtLogOn
$set     = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -RestartCount 999 -RestartInterval (New-TimeSpan -Minutes 1)
Register-ScheduledTask -TaskName "wacli-sync" -Action $action -Trigger $trigger -Settings $set -Force
```

### Layer 2: the watchdog, `%USERPROFILE%\.wacli\watchdog.ps1`

Faithful port: connection-token primary signal, kill-before-restart, store-progress backstop.

```powershell
$ErrorActionPreference = "SilentlyContinue"
$store = "$env:USERPROFILE\.wacli"
$log   = "$store\logs"; New-Item -ItemType Directory -Force -Path $log | Out-Null
$errLog = "$log\sync.err.log"; $wdLog = "$log\watchdog.log"; $stateFile = "$store\.watchdog-state.json"
$DownConfirm = 2; $StaleSeconds = 21600
function Ts { (Get-Date).ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ssZ") }
function WLog($m) { Add-Content $wdLog "$(Ts) $m" }
$state = if (Test-Path $stateFile) { Get-Content $stateFile -Raw | ConvertFrom-Json } else { @{ newest=""; observed_at=0; down_count=0 } }

function Restart-Sync($why) {
    WLog "RESTART wacli-sync: $why"
    Get-CimInstance Win32_Process -Filter "Name='wacli.exe'" |
        Where-Object { $_.CommandLine -like '*sync --follow*' } |
        ForEach-Object { Stop-Process -Id $_.ProcessId -Force }   # kill: it holds the store lock
    Start-Sleep 1
    Stop-ScheduledTask -TaskName "wacli-sync"; Start-ScheduledTask -TaskName "wacli-sync"
}

# 1. process liveness
$alive = Get-CimInstance Win32_Process -Filter "Name='wacli.exe'" | Where-Object { $_.CommandLine -like '*sync --follow*' }
if (-not $alive) { Restart-Sync "sync process not found"; @{ newest=$state.newest; observed_at=[int][double]::Parse((Get-Date -UFormat %s)); down_count=0 } | ConvertTo-Json | Set-Content $stateFile; exit }

# 2. connection token from sync.err.log (last of Connected./Disconnected./Reconnecting)
$conn = "unknown"
if (Test-Path $errLog) {
    $tok = (Select-String -Path $errLog -Pattern "Connected\.|Disconnected\.|Reconnecting" -AllMatches |
            Select-Object -Last 1).Matches.Value
    if ($tok -eq "Connected.") { $conn = "up" } elseif ($tok) { $conn = "down" }
}
$now = [int][double]::Parse((Get-Date -UFormat %s))
if ($conn -eq "down") {
    $dc = [int]$state.down_count + 1
    if ($dc -ge $DownConfirm) { Restart-Sync "connection down: reconnect loop ($dc checks)"; $dc = 0; $obs = $now }
    else { WLog "connection down (check $dc/$DownConfirm), watching"; $obs = $state.observed_at }
    @{ newest=$state.newest; observed_at=$obs; down_count=$dc } | ConvertTo-Json | Set-Content $stateFile; exit
}

# 3. store-progress backstop, only when token unknown
$newest = (& wacli --read-only --json messages list --limit 1 | ConvertFrom-Json).data.messages[0].Timestamp
if (-not $newest) { exit }
if ($newest -ne $state.newest) { @{ newest=$newest; observed_at=$now; down_count=0 } | ConvertTo-Json | Set-Content $stateFile; exit }
$stuck = $now - [int]$state.observed_at
if ($conn -eq "unknown" -and $stuck -gt $StaleSeconds) {
    Restart-Sync "backstop: no store progress ${stuck}s and token unknown"
    @{ newest=$newest; observed_at=$now; down_count=0 } | ConvertTo-Json | Set-Content $stateFile
}
```

Register it every 2 minutes:

```powershell
$action  = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-WindowStyle Hidden -ExecutionPolicy Bypass -File `"$env:USERPROFILE\.wacli\watchdog.ps1`""
$trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Minutes 2) -RepetitionDuration ([TimeSpan]::MaxValue)
Register-ScheduledTask -TaskName "wacli-watchdog" -Action $action -Trigger $trigger -Force
```

Windows ops: `Get-ScheduledTask wacli-*`, `Start-ScheduledTask wacli-sync`, `Get-Content "$env:USERPROFILE\.wacli\logs\sync.err.log" -Tail 20`.

Note: Task Scheduler runs only while the machine is on and the user session is active (the same outcome-push-during-working-hours constraint as scheduling on Windows generally). It is not a 24/7 cloud daemon. For true always-on, the layer-2 logic is identical on a Linux VM with systemd (below).

---

## Linux implementation (ported, for a VPS that needs true 24/7)

systemd `Restart=always` is layer 1; a `.timer` every 2 min runs the same watchdog (reuse the bash `watchdog.sh`, swap `launchctl kickstart` for `systemctl --user restart wacli-sync` and keep the `pkill` kill-first).

`~/.config/systemd/user/wacli-sync.service`:
```ini
[Unit]
Description=wacli sync
After=network-online.target
[Service]
ExecStart=%h/.local/bin/wacli sync --follow --download-media
Restart=always
RestartSec=30
[Install]
WantedBy=default.target
```

`~/.config/systemd/user/wacli-watchdog.service` (Type=oneshot running the bash watchdog) + `wacli-watchdog.timer` (`OnUnitActiveSec=120`). Enable with `systemctl --user enable --now wacli-sync.service wacli-watchdog.timer` and `loginctl enable-linger $USER` so it runs without an active login.

---

## Porting checklist (any OS)

1. Layer 1 keeps `wacli sync --follow --download-media` alive with a ~30s throttle.
2. Layer 2 runs every ~2 min and: checks the process is alive, reads the last connection token from `sync.err.log`, restarts after `DOWN_CONFIRM=2` consecutive downs, and **kills the wedged PID before restarting** (it holds the store lock). Store progress is a backstop only.
3. The error log is capped so a reconnect loop cannot fill the disk.
4. Every digest routine runs the liveness gate (`last_sync_at`, never `connected`) and bails loudly on a stalled sync instead of reporting "nothing happened".
5. Re-auth (`wacli auth`, QR) only when WhatsApp truly revokes the link, which is rare.
