---
name: onboard
description: Sets up a new Hebrew AIOS vault for a solo business owner and runs a warm guided interview to fill its brain. Creates the standard folder tree, copies the Hebrew CLAUDE.md kernel to the vault root plus a descriptive CLAUDE.md into every folder (Context, Projects, Daily, Resources, Skills, Intelligence, System, task-list) so Claude always knows what each folder is for, writes settings + gitignore + a tasks seed, then walks 7 simple Hebrew categories (who you are, what you sell, your ideal customer, your voice, goals, active projects, draining workflows), and builds the Context/*.md files, project folders, and first daily note from the real answers only. Use when the user says "הקמה", "התקנה", "בוא נתחיל", "onboard", "setup", or runs /onboard.
---

# Onboarding

> [!note] Sets up your vault from scratch: builds the skeleton, copies the kernel, then asks you 7 simple questions to fill the system's brain with real information about you and the business.

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. This skill interviews the client in Hebrew, and writes the client's answers into the vault files in Hebrew (the answers are Hebrew under English section headers). Keep commands, file paths, field names, dates, and numbers as they are.

This is a four-step process. Run the steps in order. Speak Hebrew, warm and simple, like a teammate. Never invent information: write only what the user actually said.

## Step 0: System mode

The system is aimed at a solo business owner or independent professional. That is the default, there is no need to choose.

Say one short sentence and that is it, do not open a mode selector. Say it in Hebrew, for example:

> אני מקים לך מערכת הפעלה אישית, מכוונת לעסק סולו. תוך כמה דקות יהיה לך מוח שזוכר הכל. קודם אני בונה את השלד, ואז נשב על 7 שאלות קצרות.

The mode is stored like this: the frontmatter of `CLAUDE.md` in the root already says `os-mode: professional`. Do not touch it.

## Step A: Build the skeleton

Create the folder tree, copy the kernel, and write the system files. Quiet work, without chattering about every step. All paths are relative to the vault root (the current working directory).

### A.1: The folder tree

```bash
mkdir -p .claude Context Projects Daily Resources Skills \
  Intelligence/meetings Intelligence/decisions Intelligence/archive \
  System/logs System/health task-list
```

### A.2: Copy the kernel (CLAUDE.md)

The kernel is written in Hebrew and ships attached to the skill, in the file `references/claude-md-kernel.md`. You copy it as-is to the vault root, named `CLAUDE.md`. Do not rewrite the kernel, only copy.

First try via `${CLAUDE_PLUGIN_ROOT}` (the path where the plugin was installed). If the variable is empty or the file is not found, there is a `find` fallback:

```bash
KERNEL=""
if [ -n "${CLAUDE_PLUGIN_ROOT:-}" ] && [ -f "${CLAUDE_PLUGIN_ROOT}/skills/onboard/references/claude-md-kernel.md" ]; then
  KERNEL="${CLAUDE_PLUGIN_ROOT}/skills/onboard/references/claude-md-kernel.md"
else
  KERNEL="$(find / -type f -path '*/onboard/references/claude-md-kernel.md' 2>/dev/null | head -1)"
fi

if [ -n "$KERNEL" ] && [ -f "$KERNEL" ]; then
  cp "$KERNEL" ./CLAUDE.md
  echo "Kernel copied to CLAUDE.md from: $KERNEL"
else
  echo "Kernel not found. Stop and ask Adir to check the plugin path."
fi
```

### A.2.1: CLAUDE.md per folder (the local context)

Every folder gets its own `CLAUDE.md` that explains what the folder is, what goes into it, and how to work in it. Claude Code loads this file automatically when working inside the folder, so the system always knows where it is and where everything is supposed to be saved. The files ship attached to the skill in `references/folder-kernels/`, you copy them as-is (same locate logic as the kernel).

```bash
KDIR=""
if [ -n "${CLAUDE_PLUGIN_ROOT:-}" ] && [ -d "${CLAUDE_PLUGIN_ROOT}/skills/onboard/references/folder-kernels" ]; then
  KDIR="${CLAUDE_PLUGIN_ROOT}/skills/onboard/references/folder-kernels"
else
  KDIR="$(find / -type d -path '*/onboard/references/folder-kernels' 2>/dev/null | head -1)"
fi

if [ -n "$KDIR" ] && [ -d "$KDIR" ]; then
  for FOLDER in Context Projects Daily Resources Skills Intelligence System task-list; do
    [ -f "$KDIR/$FOLDER.md" ] && [ -d "$FOLDER" ] && cp "$KDIR/$FOLDER.md" "$FOLDER/CLAUDE.md"
  done
  echo "CLAUDE.md embedded in folders: Context, Projects, Daily, Resources, Skills, Intelligence, System, task-list"
else
  echo "Folder kernels not found. You can add them manually later, this does not block onboarding."
fi
```

### A.3: System files

Write empty settings, a gitignore, and a seed for tasks. The tasks seed is written in Hebrew (it is a vault file the user sees), keep it as-is:

```bash
printf '{}\n' > .claude/settings.json

cat > .gitignore <<'EOF'
.DS_Store
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.env
node_modules/
System/logs/
EOF

cat > task-list/Tasks.md <<'EOF'
---
type: note
status: active
tags: [tasks, board]
---

> [!todo] לוח המשימות שלך. כל משימה חדשה נכנסת לכאן.

## משימות פתוחות

- [ ] להשלים את ההקמה ולמלא את הכספת בהקשר אמיתי ➕ TODAY
EOF
```

### A.4: Confirm the skeleton

Tell the user briefly, in Hebrew. For example:

> השלד מוכן. נוצרו: `Context/` (המוח), `Projects/`, `Daily/`, `Intelligence/`, `System/` (לוגים ובריאות), ו-`task-list/`, וכל תיקייה כוללת `CLAUDE.md` משלה שמסביר מה נכנס אליה (כך אני תמיד יודע איפה לשמור כל דבר). כדאי לפתוח את התיקייה הזו כ-Vault ב-Obsidian. עכשיו בוא נמלא אותה בך.

Continue to step B.

## Step B: The interview

A guided, simple interview in Hebrew, 7 categories. This is not an interrogation, it is a conversation. You can run it with `AskUserQuestion` (one question per category), or as simple numbered chat messages if that feels more natural. Do not use the widget. Ask the client every prompt in Hebrew, and keep their answers in Hebrew.

Before the first category say one orienting sentence in Hebrew, for example:

> 7 שאלות קצרות. אין כאן תשובות נכונות, זה פשוט מוח-דאמפ. על כל אחת אפשר לכתוב חופשי, להקליט ולהדביק תמלול, או להדביק קישורים וקבצים (פרופיל לינקדאין, עמוד אודות, מסמך, PDF). מה שיש לך ביד. אם קטגוריה לא רלוונטית, כתוב "דלג". רוצה לסיים מהר? כתוב "דלג הכל" ונבנה עם מה שיש.

Go through the categories in order. For each one: a warm, short prompt (asked in Hebrew), and always add "you can also paste links or files". Accept any format. Do not dig and do not ask follow-up questions.

The table below gives each category's prompt in Hebrew (ask it to the client verbatim in Hebrew) and where its answer is routed:

| # | Category | Warm prompt (Hebrew, ask as-is) | Routed to |
| --- | --- | --- | --- |
| 1 | Who you are and the business | ספר לי עליך: מי אתה, מה אתה עושה, איפה אתה הכי חד ביום, ולמה התחלת את מה שאתה עושה. | `Context/me.md` + `Context/business.md` |
| 2 | What you sell | מה אתה מוכר בפועל? עבור על קווי ההכנסה: לכל אחד שם, מה הוא נותן, למי, ובאיזה שלב. | `Context/services.md` (+ `business.md`) |
| 3 | The ideal customer | מי הלקוח החלום שלך? תפקיד, איך נראה היום שלו, איך הוא מתאר את הבעיה במילים שלו, ומה הוא רוצה לקבל בסוף. 3 דוגמאות אמיתיות יעזרו. | `Context/icp.md` |
| 4 | Voice and brand | איך אתה נשמע? ישיר, חם, יבש, טכני? יש ביטויים שאתה תמיד אומר, או מילים שלא תיגע בהן? **הדבק דוגמת כתיבה אמיתית שלך** (פסקה או קישור) ואלמד ממנה את הקול. | `Context/brand.md` |
| 5 | Goals and priorities | לאן אתה הולך השנה? 1 עד 3 יעדים עם מספר (הכנסה, קהל, תאריך), למה הם חשובים, ולמה אתה אומר לא כדי להתמקד. | `Context/strategy.md` |
| 6 | Active projects | על מה אתה עובד עכשיו? לכל פרויקט: שם, מטרה במשפט, סטטוס, דדליין אם יש, ומי עוד מעורב. | `Projects/{name}/` |
| 7 | Tools and the draining workflows | באילו כלים אתה עובד, ואיפה האמת חיה לכל תהליך? והכי חשוב: מהן 1 עד 2 העבודות החוזרות שהכי מתישות אותך? (כשקורה X אני עושה Y, לוקח Z, ומה שהייתי רוצה זה V). אלה המועמדות הראשונות לאוטומציה. | `Context/infrastructure.md` + `Context/pain-points.md` |

Answer-collection rules:
- A text or transcript answer: save raw, do not rephrase.
- A pasted link: pull it with `defuddle parse <url> --md`, and if that fails, with `WebFetch`.
- A local file path: read it with `Read`. A folder: go over it with `Glob` and read each file.
- "דלג" (skip): skip the category. "דלג הכל" (skip all): stop the interview and go to step C with what was already collected.

Gather everything into a corpus by category, then move to building. No mid-stream summaries.

## Step C: Build

Now build the files from the answers. Quiet work. In every file write only real information the user gave. The content written into these vault files is in Hebrew (the client's own words).

### The build rule (critical)

The files in `references/` are **skeletons**, not deliverables. They show the section structure only.

1. Read the relevant skeleton to learn the structure (title, sections, order).
2. Replace every placeholder in brackets `[...]` with a real value from the answers and the corpus.
3. A section with no information at all: **drop the whole section**. Never leave `[brackets]`, "TBD", or template text.
4. A section with partial information: keep it, delete only the empty bullets.
5. Use the user's exact words, names, and numbers. Do not rephrase facts.
6. In each file's frontmatter: `date:` = today's date.
7. Do not create an empty context file. If a category had no information, do not create its file.

### Mapping a context file to its skeleton

| Output file | Source skeleton | Created only if |
| --- | --- | --- |
| `Context/me.md` | `references/context-me.md` | always (there is info from category 1) |
| `Context/business.md` | `references/context-business.md` | category 1 or 2 had info about the business |
| `Context/services.md` | `references/context-services.md` | category 2 had info |
| `Context/icp.md` | `references/context-icp.md` | category 3 had info |
| `Context/brand.md` | `references/context-brand.md` | category 4 had info |
| `Context/strategy.md` | `references/context-strategy.md` | category 5 had info |
| `Context/infrastructure.md` | `references/context-infrastructure.md` | category 7 had tools |
| `Context/pain-points.md` | `references/context-pain-points.md` | category 7 had draining workflows |

### Projects

For every project mentioned in category 6, create `Projects/{name}/README.md`. The folder name in kebab-case. The template (the content is filled in Hebrew from the answers):

```markdown
---
type: project
date: YYYY-MM-DD
status: active
tags: [project]
---

> [!note] [the project goal in one sentence]

## סטטוס
[where things stand]

## הצעדים הבאים
[what needs to happen]
```

Fill only from what was given. A simple mention ("working on a podcast") = README only. Do not create empty subfolders. Use `[[wikilinks]]` for every person, company, or project mentioned.

### The first daily note

Create `Daily/YYYY-MM-DD.md` (today's date). The content is written in Hebrew:

```markdown
---
type: daily-note
date: YYYY-MM-DD
status: active
tags: [daily]
---

## הסשן
- **מיקוד**: הקמת הכספת ותשאול ראשוני.
- **הושלם**: בניית השלד והקמת המוח מתוך התשאול.
- **הצעדים הבאים**: [according to what came up in the conversation]
```

### Confirmation

Tell the user briefly, in Hebrew: which context files were created, how many projects, and that they can add context at any moment ("just tell me and I will update the right file"). Offer one next step based on what they shared.

## Rules

1. Speak Hebrew, warm and simple. Like a teammate, not like an AI.
2. Never invent information. Write only what the user actually said.
3. A skeleton with `[brackets]` is a skeleton, not a deliverable. Drop every section without information. Do not leave a placeholder or "TBD".
4. Do not create an empty context file.
5. Do not touch the kernel. You copy `claude-md-kernel.md`, you do not rewrite it.
6. Every file lives in an existing folder. Never in the root (except `CLAUDE.md`).
7. `[[wikilinks]]` for every entity in the vault files.
8. Never use an em dash. Comma, period, colon, or split into sentences.
9. Accept any format: text, transcript, links, files, or "דלג". Do not dig.
10. Quiet work. Build, then summarize briefly. Do not chatter about every file.

## Troubleshooting

- **The kernel was not found**: if both `${CLAUDE_PLUGIN_ROOT}` and the `find` failed, stop. Do not write `CLAUDE.md` from memory. Ask Adir to check that the `aios-core` plugin is installed.
- **A link did not pull**: if `defuddle` and `WebFetch` both failed, say so quietly and continue with the rest of the answers.
- **The user wrote "דלג הכל"**: skip straight to step C and build with what there is. Do not create empty files.
