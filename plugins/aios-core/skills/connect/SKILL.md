---
name: connect
description: "Wires the AIOS vault to the outside world step by step, in plain Hebrew, for a non-technical business owner. Connects Google (Gmail, Drive, Calendar) through the gws CLI with a guided OAuth flow (create a Desktop OAuth client in Google Cloud Console, download the secret JSON into ~/.config/gws, run gws auth login, enable the Gmail/Drive/Calendar APIs, then verify with a real call), connects WhatsApp through wacli (QR login + wacli doctor verify), and adds a generic data-source API key slot (e.g. GovMap) into a secrets file with a verify call. After each connection succeeds it marks that line in System/health/status.md with a ✅ and a timestamp. Use when the user says 'חיבור', 'תחבר', 'לחבר מערכות', 'connect', 'חבר ג'ימייל', 'חבר וואטסאפ', or runs /connect."
---

# Connect systems (Connect)

> [!note] Connects your system to the world: Gmail, Drive and Calendar (through gws), WhatsApp (through wacli), and a data-source API key. Each connection is a short, clear step, and after each one works we mark it ✅ in `System/health/status.md`.

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

This is where the system stops being isolated and starts seeing your world. First we make sure all the tools you need are installed on the computer (**Step 0**), and only then we connect: **Google** (mail, files, calendar), **WhatsApp**, and a **data source** with an API key. We do this together, step by step, and we do not move to the next connection before the current one really works.

Assume this is a brand-new computer, and that this may be the first time you touch a terminal. That is perfectly fine. We start from zero, and I will check on my own what is missing and walk you through the installation.

Before you start, say one orienting sentence:

> Let's connect the system to your world. We'll do it together, piece by piece. First I check that all the tools are installed on the computer, and if something is missing I give you exactly what to click. Then we connect Google, WhatsApp and a data source, and at the end of each connection I check that it really works and mark it as healthy. If something gets stuck, there are fixes for the common issues below.

Always start at **Step 0** (tool check). Without the tools no connection will work, so it is not a step to skip. Once the tools are installed, you can run with `AskUserQuestion` (Google / WhatsApp / data source / everything), or simply start with Google if the user says "let's go". Each connection stands on its own, you can do just one and come back to the rest later.

Note: all commands run from inside the vault folder (the working directory). The connection state is written to `System/health/status.md` inside the vault.

The system works on three operating systems: **macOS** (Mac), **Windows**, and **Linux**. In Step 0 we auto-detect which one you are on, and give installation instructions exactly for yours. On Windows all the later commands run inside **Git Bash** (a small program installed in Step 0), and gws stores its config under the user's home folder, so `~/.config/gws` resolves correctly from Git Bash too.

## Step 0: Tool check and installation

Before any connection, we make sure all the tools the system needs are actually installed on the computer. Assume this is a new computer with nothing on it yet. Don't worry, it is not complicated, and I am with you at every step.

> This is a one-time moment. We just install the basic tools the computer needs to talk to Google and WhatsApp, once. Adir is doing this with you now, and after this time you do not need to repeat it. If you see strange words, ignore them and just follow what I tell you to click.

The tools you need:

| Tool | What for | Required? |
| --- | --- | --- |
| Node.js | Runs gws (the Google connection) | Required |
| npm | Installs gws, comes together with Node.js | Required |
| git | Brings and updates the system, and on Windows also runs the commands | Required |
| gws | The tool that connects Gmail, Drive and Calendar | Required |
| wacli | The WhatsApp connection | Optional, works on Mac, Windows and Linux |

### 0.1: Detect the operating system and check the tools

Run this block. It detects on its own whether you are on Mac, Windows or Linux, and goes tool by tool checking what is already installed and what is missing. At the end you get a clear list with ✅ next to what you have and ❌ next to what is missing.

```bash
case "$(uname -s)" in
  Darwin) OS=mac ;;
  Linux) OS=linux ;;
  MINGW*|MSYS*|CYGWIN*) OS=windows ;;
  *) OS=unknown ;;
esac

case "$OS" in
  mac) echo "💻 זוהתה מערכת הפעלה: macOS (מק)" ;;
  windows) echo "💻 זוהתה מערכת הפעלה: Windows (חלונות), הפקודות רצות ב-Git Bash" ;;
  linux) echo "💻 זוהתה מערכת הפעלה: Linux (לינוקס)" ;;
  *) echo "💻 לא הצלחתי לזהות את מערכת ההפעלה. נמשיך בזהירות." ;;
esac

echo ""
echo "בודק כלים:"
echo "--------------------------------"

if command -v node >/dev/null 2>&1; then
  echo "✅ Node.js מותקן ($(node -v))"
else
  echo "❌ Node.js חסר"
fi

if command -v npm >/dev/null 2>&1; then
  echo "✅ npm מותקן ($(npm -v))"
else
  echo "❌ npm חסר (מגיע יחד עם Node.js)"
fi

if command -v git >/dev/null 2>&1; then
  echo "✅ git מותקן ($(git --version))"
else
  echo "❌ git חסר"
fi

if command -v gws >/dev/null 2>&1; then
  echo "✅ gws מותקן ($(gws --version 2>/dev/null || echo 'גרסה לא ידועה'))"
else
  echo "❌ gws חסר (מתקינים אחרי ש-Node.js קיים)"
fi

if command -v wacli >/dev/null 2>&1; then
  echo "✅ wacli מותקן (וואטסאפ)"
else
  echo "➖ wacli עדיין לא מותקן (אופציונלי, מתקינים אותו בשלב 2 כשמחברים וואטסאפ)"
fi
echo "--------------------------------"
```

Read the list together with the user. Anything marked ❌ needs to be installed per the next section. If everything is ✅ (except wacli, which is optional), you can skip straight to **Step 1**.

### 0.2: Install what is missing, per your operating system

Give installation instructions **only** for the tools that came out ❌, and **only** for the operating system detected above. Explain in plain Hebrew, give one step and wait for the user to finish before you continue.

> We'll go in order: first git, then Node.js, and finally gws (because gws needs Node.js to already be there). After each install, **it is important to open a new terminal** so the computer "sees" the new tool.

**git (missing?)**

- **Mac:** the simplest way is through Apple's developer tools. Run `xcode-select --install`, a window opens, click **Install** and wait for it to finish. (If Homebrew is installed, you can use `brew install git` instead.)
- **Windows:** download "Git for Windows" from `https://git-scm.com/download/win`, run the file, and click **Next** all the way through (leave all the defaults, do not change anything). This also installs **Git Bash**, the program inside which all our commands run on Windows. After installation, open **Git Bash** from the Start menu and continue from there.
- **Linux:** install through your distribution's package manager, e.g. `sudo apt install git` (Ubuntu/Debian) or `sudo dnf install git` (Fedora).

```bash
git --version
```

**Node.js (missing?)** brings npm with it.

- **Mac:** go to `https://nodejs.org`, download the **LTS** version (the left, stable button), run the file and click Continue all the way through. (If Homebrew is installed, you can use `brew install node`.)
- **Windows:** go to `https://nodejs.org`, download the **LTS** version, run the file and click Next all the way through (defaults). Alternatively, for those who know it: `winget install OpenJS.NodeJS.LTS`.
- **Linux:** install through the package manager (`sudo apt install nodejs npm`), or download from `https://nodejs.org` if the distribution gives an old version.

```bash
node -v
npm -v
```

> We need Node version 18 or higher (we work with version 20). If `node -v` shows a number lower than 18, reinstall the LTS version from the site.

**gws (missing?)** install only after `node -v` and `npm -v` work. The installation is identical on all systems:

```bash
npm install -g @googleworkspace/cli
```

Then verify:

```bash
gws --version
```

> If it shows a version number, gws is installed. If it says "permission denied" on Linux/Mac, try again with `sudo npm install -g @googleworkspace/cli`.

**wacli (WhatsApp)** is the tool that connects WhatsApp, and it works on **all three operating systems**: Mac, Windows and Linux. You do not need to install it now, we do that inside Step 2 (WhatsApp connection), where there are exact installation instructions for each system. If you want to skip WhatsApp for now, no problem: the morning report, the supervisor and the health-ping work great without WhatsApp too (they write to the vault), and WhatsApp will connect any moment you want.

### 0.3: Re-check after installation

After you have installed what was missing, **open a new terminal** (on Windows: a new Git Bash window) and run this short block again. Every line should show ✅. If something is still ❌, see `## Troubleshooting`.

```bash
case "$(uname -s)" in
  Darwin) OS=mac ;;
  Linux) OS=linux ;;
  MINGW*|MSYS*|CYGWIN*) OS=windows ;;
  *) OS=unknown ;;
esac
ALL_OK=1
for tool in node npm git gws; do
  if command -v "$tool" >/dev/null 2>&1; then
    echo "✅ $tool"
  else
    echo "❌ $tool עדיין חסר"
    ALL_OK=0
  fi
done
if [ "$ALL_OK" = "1" ]; then
  echo "🎉 כל הכלים מוכנים. אפשר להתחיל לחבר (שלב 1)."
else
  echo "עוד חסר משהו. התקן אותו, פתח טרמינל חדש, והרץ שוב את הבלוק הזה."
fi
```

When all the required tools (`node`, `npm`, `git`, `gws`) are marked ✅, **Step 0 is done** and you can move to the Google connection in Step 1.

## Step 1: Connect Google (Gmail, Drive, Calendar)

This is the most useful and longest connection, five small steps. We give the system permission to read your mail, files and calendar, through a tool called `gws`. Behind the scenes this is an official Google authorization process (called OAuth), but you only click a few buttons and paste one file.

> We are building your own personal "entry key" at Google, that lets the system read Gmail, Drive and Calendar on your behalf. Google requires you to create it yourself so control stays with you. I walk you through it step by step. We'll take it slow.

### 1.1: Create a project and identify with Google (Google Cloud Console)

Explain to the user in plain Hebrew and ask them to do these steps in the browser. Don't rush, give one step and wait.

> 1. Go to: `https://console.cloud.google.com` and sign in with the Google account whose mail you want to connect.
> 2. At the top, next to the logo, there is a project picker. Click it, then "New Project". Give it a simple name like `aios` and click Create. Wait a few seconds for it to be created, then make sure it is the selected project at the top.
> 3. In the side menu (the three lines at the top left) click **APIs & Services**, then **OAuth consent screen**. If this is the first step, choose **External**, fill in only the fields with an asterisk (app name, your email), and save. You can skip the non-required fields.
> 4. Still on the consent screen, scroll to **Test users** and click **Add users**. **Add your own Gmail address there.** This is critical. Without it the sign-in will fail later. Save.

> [!important] Most of the trouble at this step is because the user was not added as a Test user, or the app is in "Testing" and they did not add themselves. If that happened, there is a full fix in `## Troubleshooting` below.

### 1.2: Create the entry key (OAuth client) and download the file

> 5. In the side menu: **APIs & Services**, then **Credentials**.
> 6. At the top click **Create Credentials**, and choose **OAuth client ID**.
> 7. In the **Application type** field choose **Desktop app**. Give it a name like `aios-desktop` and click **Create**.
> 8. A window opens with the details. Click **Download JSON** and save the file. This is your "entry key". Do not share it with anyone.

Get the path to the downloaded file from the user (usually in the Downloads folder), and move it to where `gws` looks for it. Create the folder if there is none, and put the file under the name `client_secret.json`:

```bash
mkdir -p ~/.config/gws
# החלף את הנתיב בקובץ שהמשתמש הוריד (בדרך כלל ~/Downloads/client_secret_*.json)
cp ~/Downloads/client_secret_*.json ~/.config/gws/client_secret.json
echo "מפתח הכניסה הועתק אל ~/.config/gws/client_secret.json"
```

If the user is not sure where the file is, search for it together:

```bash
ls -t ~/Downloads/client_secret_*.json 2>/dev/null | head -1
```

### 1.3: Sign in (gws auth login)

Now we connect. This command opens a browser, you approve with your Google account, and that's it.

```bash
gws auth login
```

> In the browser that opens: choose your Google account, and if Google warns that "the app is not verified" click **Advanced** then **Go to aios (unsafe)**. This is fine: it is the app you created yourself a moment ago. Approve the permissions.

After the browser says the sign-in succeeded, check that the state is saved:

```bash
gws auth status
```

If it shows a connected account, great. If it fails, see `## Troubleshooting`.

### 1.4: Enable the services (Gmail, Drive, Calendar)

The sign-in gave an identity. Now we need to "turn on" the three services we want, in the project you created. The easiest way is through the browser, a direct link for each service:

> Open each of these links (make sure the right project is selected at the top), and click **Enable** in each:
> - Gmail: `https://console.cloud.google.com/apis/library/gmail.googleapis.com`
> - Drive: `https://console.cloud.google.com/apis/library/drive.googleapis.com`
> - Calendar: `https://console.cloud.google.com/apis/library/calendar-json.googleapis.com`

If the user has the `gcloud` tool installed, you can turn all three on in a single command instead of manually:

```bash
gcloud services enable gmail.googleapis.com drive.googleapis.com calendar-json.googleapis.com 2>/dev/null \
  && echo "שלושת השירותים הופעלו" \
  || echo "אין gcloud או שצריך לבחור פרויקט. אין בעיה, הדלק דרך הקישורים בדפדפן."
```

### 1.5: A real check (verify)

We make sure the connection really works: we ask gws for one file from Drive.

```bash
gws drive files list --params '{"pageSize": 1}' --format json 2>&1 | head -20
```

If it returned info about a file (or an empty list with no error), **Google is connected**. Go to the connection logging below (Step 4). If it returned an error about a service that was not enabled, wait a minute (enabling takes time to propagate) and try again, or see `## Troubleshooting`.

Once the check passes, mark Google as healthy: go to **Step 4** and update the three rows (Gmail, Drive, Calendar) in `status.md`.

## Step 2: Connect WhatsApp (wacli)

> [!note] Works on every operating system. WhatsApp connects through a tool called `wacli`, which has a version for Mac, Windows and Linux. First we install the tool for your system (2.1), then we connect by scanning a QR code (2.2), and we verify everything is healthy (2.3).

The WhatsApp connection is short: you install a small tool once, then scan a QR code from your phone, exactly like WhatsApp Web.

> We'll connect your WhatsApp so the system can read and send messages on your behalf. First I install a small tool called wacli (once), and then it is exactly like connecting WhatsApp Web: you scan a code from your phone.

### 2.1: Install wacli (per your operating system)

First we detect again which system you are on and check whether `wacli` is already installed (it may already have come out ✅ in Step 0). If it is already installed, skip straight to 2.2.

```bash
case "$(uname -s)" in
  Darwin) OS=mac ;;
  Linux) OS=linux ;;
  MINGW*|MSYS*|CYGWIN*) OS=windows ;;
  *) OS=unknown ;;
esac

if command -v wacli >/dev/null 2>&1; then
  echo "✅ wacli כבר מותקן. אפשר לדלג ל-2.2 (התחברות)."
else
  echo "➖ wacli עדיין לא מותקן. מתקינים אותו עכשיו לפי המערכת: $OS"
fi
```

Give installation instructions **only** for the operating system detected:

**Mac:** installation through Homebrew (the Mac package manager). Run:

```bash
brew install openclaw/tap/wacli
```

> If the command says `brew` does not exist, install Homebrew once from the site `https://brew.sh` (there is a single install line to copy there), open a new terminal, and run the `brew install` line above again.

**Windows:** you download a ready-made file from wacli's site and put it where the computer finds it. If the `gh` tool (GitHub CLI) is installed, this is fully automatic:

```bash
# הדרך האוטומטית (אם מותקן gh): מוריד את הגרסה האחרונה לחלונות ופותח אותה
gh release download -R openclaw/wacli -p '*windows_amd64.zip' --clobber \
  && unzip -o wacli_*_windows_amd64.zip -d "$HOME/bin" \
  && echo "wacli.exe הותקן בתיקייה $HOME/bin"
```

> If there is no `gh`, do it manually: go to `https://github.com/openclaw/wacli/releases/latest`, download the file whose name ends in `windows_amd64.zip` (e.g. `wacli_0.11.1_windows_amd64.zip`), unzip it, and take `wacli.exe` out of it. Put `wacli.exe` in a folder that is on the PATH, e.g. `C:\Users\<name>\bin`. Inside Git Bash that folder is `$HOME/bin`, and you can create it and add it to the PATH like this:

```bash
# פעם אחת בחלונות (Git Bash): יוצר תיקיית bin אישית ומוסיף אותה ל-PATH
mkdir -p "$HOME/bin"
echo 'export PATH="$HOME/bin:$PATH"' >> "$HOME/.bashrc"
echo "אחרי שתשים את wacli.exe בתוך $HOME/bin, פתח חלון Git Bash חדש."
```

**Linux:** simplest through Homebrew if it is installed, otherwise download a ready-made file:

```bash
# אם מותקן Homebrew על לינוקס:
brew install openclaw/tap/wacli
```

> If there is no Homebrew on Linux: go to `https://github.com/openclaw/wacli/releases/latest`, download the file whose name ends in `linux_amd64.tar.gz`, extract it (`tar -xzf wacli_*_linux_amd64.tar.gz`), and move the resulting `wacli` file to a folder on the PATH (e.g. `sudo mv wacli /usr/local/bin/`). Anyone who has `gh` installed can use it here too: `gh release download -R openclaw/wacli -p '*linux_amd64.tar.gz' --clobber`.

After installation, **open a new terminal** (on Windows: a new Git Bash window) and make sure `wacli` is found:

```bash
command -v wacli >/dev/null 2>&1 && echo "✅ wacli מותקן ומוכן" || echo "❌ wacli עדיין לא נמצא. ראה פתרון תקלות."
```

> If it still shows ❌ after a new terminal, the file is probably not in a folder on the PATH. See `## Troubleshooting`.

### 2.2: Sign in and scan the QR

```bash
wacli auth
```

> A QR code will appear in the terminal. On the phone: open WhatsApp, go to **Settings**, **Linked devices**, **Link a device**, and scan the code on the screen. After scanning, wait a few seconds until it says the sign-in succeeded.

If the user prefers to sign in with a phone number instead of scanning, you can:

```bash
# החלף במספר הבינלאומי המלא, למשל 9725XXXXXXXX
wacli auth --phone 9725XXXXXXXX
```

### 2.3: Check (wacli doctor)

We run wacli's diagnostic to make sure everything is connected and healthy.

```bash
wacli doctor
```

> You should see that the store is healthy, that auth is active, and that search works. If something is marked as problematic, see `## Troubleshooting`.

If `wacli doctor` looks healthy, **WhatsApp is connected**. Now we make sure it stays connected forever (2.4), before we mark it as healthy.

### 2.4: Keep the connection alive (so it always works and survives crashes)

> [!important] This is what turns WhatsApp from "connected now" into "always connected"
> The WhatsApp sync can drop silently: the process stays alive but the connection is dead (usually after the computer sleeps and wakes), and it gets stuck in a sign-in loop without recovering on its own. To prevent this, we install two protective layers that run on their own in the background: a layer that brings the sync back up if it drops, and a watchdog that every two minutes detects a silently-dead connection and force-restarts it.

This is a setup step that Adir runs once, per operating system. The exact instructions and the ready-made files (LaunchAgents for Mac, Task Scheduler with PowerShell for Windows, systemd for Linux), including the full watchdog script, are in `references/wacli-always-on.md`. One principle you must not miss: **never trust the `connected` field of `wacli doctor` as a liveness signal.** When the sync is running it holds the store lock, and then `connected` is always `false` regardless of the real state. The reliable signal is `last_sync_at` and the connection state in the sync's log.

After installation, make sure both layers came up (on Mac: `launchctl list | grep wacli`, you should see `sh.wacli.sync` and `sh.wacli.watchdog`), and then go to **Step 4** and mark the WhatsApp row in `status.md` as "connected, protection active".

## Step 3: Connect a data source (API key, e.g. GovMap)

Sometimes the system needs an external data source: a map, a registry, or any service that gives you an API key. The example here is **GovMap** (maps and parcels in Israel), but the same principle fits any key: you store it in a single secrets file, and you do not scatter it.

> Do you have a data source with an API key? We'll store it in one safe place inside the vault, in a secrets file that does not go into the backup. Bring the key (you usually get it on the service's site, in an "API" or "keys" area), and I'll paste it in the right place.

### 3.1: Store the key in the secrets file

The secrets file is `.env` in the vault root. It is already excluded from the backup (the setup's `.gitignore` includes `.env`), so the key will not be exposed. Add the key without overwriting existing keys:

```bash
# החלף GOVMAP_API_KEY בשם המקור, ו-PASTE_KEY_HERE במפתח האמיתי שהמשתמש נתן
touch .env
chmod 600 .env
if grep -q '^GOVMAP_API_KEY=' .env 2>/dev/null; then
  echo "כבר קיים מפתח GOVMAP_API_KEY בקובץ. לא דורסים. אם צריך לעדכן, ערוך ידנית."
else
  printf 'GOVMAP_API_KEY=%s\n' 'PASTE_KEY_HERE' >> .env
  echo "המפתח נשמר ב-.env בשורש הכספת (מוחרג מגיבוי)."
fi
```

Also record in `Context/infrastructure.md` that this data source is connected (which service, and what it is used for), without writing the key itself. The key name yes, the value never.

### 3.2: Check (verify)

Load the key and check that it is read and that the service responds. Each service has its own check URL, adapt the `curl` to the service the user is connecting. A generic example:

```bash
set -a; . ./.env; set +a
if [ -z "$GOVMAP_API_KEY" ] || [ "$GOVMAP_API_KEY" = "PASTE_KEY_HERE" ]; then
  echo "❌ המפתח לא הודבק עדיין. חזור ל-3.1 והדבק את המפתח האמיתי."
else
  echo "✅ המפתח נטען (${#GOVMAP_API_KEY} תווים). עכשיו נבדוק מול השירות."
  # החלף בכתובת הבדיקה של השירות בפועל. דוגמה כללית:
  # curl -s -o /dev/null -w '%{http_code}\n' -H "Authorization: Bearer $GOVMAP_API_KEY" "https://API_BASE/health"
fi
```

> If the check returned a healthy response (code 200, or info from the service), **the data source is connected**. If it returned a permission error (401/403), the key is probably wrong or inactive, check it against the service's site.

If the check passed, go to **Step 4** and mark the data-source row in `status.md`.

## Step 4: Log the connection in status.md

After a connection passes its check, we update `System/health/status.md` so that both you and `/doctor` always know what is connected. This is the home of the system state.

If the file does not exist yet, create it with this skeleton (once):

```bash
mkdir -p System/health
if [ ! -f System/health/status.md ]; then
cat > System/health/status.md <<'EOF'
---
type: health
status: active
tags: [health, status, connections]
---

> [!note] מצב המערכת: מה מחובר ומה רץ. מתעדכן בכל `/connect` ובכל `/doctor`.

## חיבורים

| מערכת | מצב | עודכן |
| --- | --- | --- |
| ג'ימייל (gws) | ⬜ לא מחובר | - |
| דרייב (gws) | ⬜ לא מחובר | - |
| יומן (gws) | ⬜ לא מחובר | - |
| וואטסאפ (wacli) | ⬜ לא מחובר | - |
| מקור נתונים (API) | ⬜ לא מחובר | - |

## שגרות

| שגרה | ריצה אחרונה | מצב אחרון | תזמון |
| --- | --- | --- | --- |
| morning-report | - | - | - |
EOF
echo "נוצר System/health/status.md"
fi
```

Now update the rows for what was just connected. Replace the cell with the ✅ מחובר state and the current timestamp. For example, after Google passed, update the three rows for Gmail, Drive and Calendar:

```bash
NOW=$(date '+%Y-%m-%d %H:%M')
echo "סמן עכשיו ✅ מחובר + הזמן $NOW בשורות הרלוונטיות ב-System/health/status.md"
```

Edit `System/health/status.md` for real (with the editing tool, not necessarily through bash): in every row that was connected, change the **state** column to `✅ מחובר` and the **updated** column to the `$NOW` timestamp. Leave the rows that are not connected yet as they are (`⬜ לא מחובר`).

Also add a log line for the run, like every action in the system:

```bash
DATE=$(date +%F); TIME=$(date +%H:%M); mkdir -p System/logs
printf -- '- %s | connect | ✅ הצליח | חובר: %s | משך: %ss\n' \
  "$TIME" "גוגל" "$DUR" >> "System/logs/$DATE.md"
```

(Replace `"גוגל"` with what was actually connected in this run, and `$DUR` with the duration in seconds. If one connection failed and another succeeded, log ⚠️ חלקי and note what worked and what did not.)

## Step 5: Summary

In the chat, briefly say what is connected now, what is left to connect, and what you can do with it. Specific, not generic:

> Gmail, Drive and Calendar are connected and checked. WhatsApp not yet, we'll do it when you want. Now you can run `/morning-report` to get a morning report on your mail and calendar. For a full check of the system state any time: `/doctor`.

WhatsApp works on every operating system, so you can suggest it as the next step to any user (Mac, Windows or Linux) if it is not connected yet. Suggest only one next step, based on what is already connected.

## Rules

1. Always start at **Step 0** (tool check). Do not attempt any connection before every required tool (`node`, `npm`, `git`, `gws`) is marked ✅. If a tool is missing, give exact installation instructions only for the missing tool and only for the detected operating system, and stop until the user installs it.
2. Detect the operating system through the `uname` block in Step 0, and always match the instructions (a command or a download link) to the detected system. Do not assume Mac. On Windows the commands run in Git Bash.
3. wacli works on every operating system (Mac, Windows, Linux). Install it per the detected system: Mac with `brew install openclaw/tap/wacli`, Windows by downloading the zip from GitHub releases (or `gh release download`) and putting `wacli.exe` on the PATH, Linux with brew or downloading the tar.gz. If the user chooses to skip WhatsApp, the system keeps working (writing to the vault).
4. One connection at a time. Do not move to the next connection before the current one passed a real check.
5. Every connection must have a real **verify check** (a gws read of Drive, `wacli doctor`, or an API call). A connection without a check does not count as connected.
6. After a check passes, immediately update `System/health/status.md` (✅ state + timestamp) and add a log line to `System/logs/`.
7. Keys and secrets live in `.env` in the vault root only, with `600` permissions. Never write a key value into a regular vault file, into `status.md`, or into the chat. The key name yes, the value no.
8. Do not overwrite an existing key in `.env`. If one already exists, say so and leave it.
9. Speak plain, reassuring Hebrew. The user is non-technical. Explain each step in human language, give one step and wait for a reply.
10. Do not rush the user inside the Cloud Console. Step by step, and always explain *why* (this is your authorization, control stays with you).
11. If a connection fails, do not continue as if it succeeded. Point to `## Troubleshooting`, and update `status.md` as `⚠️` or `❌` accordingly.
12. `[[wikilinks]]` for every system or service mentioned in the vault files (e.g. in `Context/infrastructure.md`).
13. Never use an em dash. Comma, period, colon, or split into sentences.

## Troubleshooting

- **I installed a tool (node / git / gws) but the check still says "not found" / "command not found"**: almost always this is because the current terminal was opened before the installation and therefore does not yet "see" the new tool. **Close the terminal and open a new window** (on Windows: a new Git Bash window), and run the re-check (Step 0.3) again. That refreshes the PATH and solves most cases. If it is still missing even after a new terminal, the installation probably did not finish, repeat Step 0.2 for that tool.
- **`gws auth login` fails or opens an "Access blocked" / "app is in testing" window**: your app is in "Testing" mode, and you need to add yourself as a test user. Go back to Google Cloud Console, **APIs & Services**, **OAuth consent screen**, **Test users**, **Add users**, and add your Gmail address. Save and try `gws auth login` again. This is the most common issue.
- **Google warns "Google hasn't verified this app"**: this is fine and not blocking. Click **Advanced** then **Go to ... (unsafe)**. This is the app you created, it is safe.
- **The `gws drive files list` check returns an error about a service that is not enabled (API not enabled / SERVICE_DISABLED)**: you did not enable the service, or it just happened and the change has not propagated yet. Make sure you enabled Gmail, Drive and Calendar (Step 1.4), wait a minute, and try again.
- **`cp ~/Downloads/client_secret_*.json` did not find a file**: the file downloaded to a different place, or under a different name. Run `ls -t ~/Downloads/*.json | head -3` to find it, and use the right path.
- **You accidentally chose "Web application" instead of "Desktop app"**: create a new OAuth client of type **Desktop app**, download the new JSON, and copy it again to `~/.config/gws/client_secret.json` (overwriting the old one). Sign in again.
- **I installed wacli but the check says "wacli עדיין לא נמצא"** (Windows or Linux, manual install): the file is probably not in a folder on the PATH. On Windows, make sure `wacli.exe` sits inside the folder you added to the PATH (e.g. `$HOME/bin`), that you added the line to `~/.bashrc`, **and opened a new Git Bash window**. On Linux, make sure you moved `wacli` to a folder like `/usr/local/bin` and that it has execute permission (`chmod +x`). You can always run it with the full path to check that the file itself works.
- **`brew install openclaw/tap/wacli` says brew does not exist** (Mac or Linux): Homebrew is not installed. Install it once from `https://brew.sh` (a single install line to copy), open a new terminal, and run the `brew install` line again.
- **The WhatsApp QR code expired before you managed to scan it**: this is fine, the code refreshes. Run `wacli auth` again and scan faster. If the terminal does not display the code nicely, add `--qr-format text`.
- **`wacli doctor` marks the store or search as problematic after a fresh sign-in**: sometimes the initial sync is still running. Wait a minute and try again. If it stays stuck, run `wacli auth --follow` to let the sync continue, then `wacli doctor` again.
- **The API key check returns 401 or 403**: the key is wrong, expired, or not enabled on the service side. Check the value against the service's site, make sure there are no extra spaces, and update `.env` (edit it manually so you do not duplicate the line).
