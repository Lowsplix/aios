---
name: level-up
description: "Finds the next thing to automate using Nate Herk's 5-question pattern in Hebrew (what you did 3+ times this week, what felt manual/boring/copy-paste, what a smart assistant could have done, what breaks first at 500 customers, what would give you 500 customers if it ran alone). From the answers, proposes 1-3 concrete skills (name, trigger, one-line goal, steps), lets the user pick one, then scaffolds a Skills/{name}/SKILL.md stub in Hebrew following the AIOS conventions. Use when the user says 'level up', 'שדרוג', 'מה לבנות הבא', 'מה לאוטמט', 'תבנה לי מיומנות', or runs /level-up."
---

# Level up (the next skill)

> [!note] Finds the next thing worth automating, through five questions. From the answers it proposes 1 to 3 concrete skills, you pick one, and it gets built as a ready `SKILL.md` skeleton in the vault.

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

This is the natural follow-up to `/audit`. The audit finds where the system is weak (which C is low). Level up takes it one step further: instead of another report, it pulls out of you the single most worthwhile thing to build, and starts building it. The base is Nate Herk's five-question pattern for spotting automations.

## Step 1: Quick background (optional, recommended)

If `/audit` just ran, glance at the latest report in `System/health/` to see which C is lowest. That steers the questions: if cadence is low, look for a routine to build. If capability is low, look for a repetitive action to automate. Also briefly read `Context/me.md` and `Context/services.md` so the proposals fit the business and are not generic.

## Step 2: The five questions

Ask the five questions, one by one or together, in plain Hebrew. Let the business owner answer in their own words. These are the questions that pull out the right automation:

1. **What did you do three or more times this week?** (Repetition is the strongest signal for automation.)
2. **What felt manual, boring, or copy-paste?** (Friction equals opportunity.)
3. **What could a "smart assistant" have done for you, but you did yourself?** (Work that does not actually require you.)
4. **If 500 customers walked in tomorrow, what would break first?** (The bottleneck that limits growth.)
5. **What would give you 500 customers if it ran on its own?** (The unused lever.)

Listen for patterns. Questions 1 to 3 find existing waste. Questions 4 and 5 find the limit to growth. The gold is where an answer recurs across several questions.

## Step 3: Propose 1 to 3 skills

From the answers, propose 1 to 3 concrete skills. Not vague ideas, skills you can build. Each one in exactly this format:

| Field | What to write |
| --- | --- |
| **Name** | kebab-case, becomes a command. For example `lead-followup`. |
| **Trigger** | What sets it off: a command, a Hebrew word, or a schedule. |
| **Goal in one sentence** | What it does, in one line. |
| **Steps** | 3 to 5 steps it performs, briefly. |

Tie each proposal to the answer it came from ("you said you build a price quote manually every week, so..."). Rank from most impactful to least, as in `/audit`: what frees the most time or removes the biggest bottleneck.

Show the proposals and find out which one to pick using `AskUserQuestion`. Do not build before they pick. If no proposal is exact, sharpen it with one question and try again.

## Step 4: Build the skeleton of the chosen skill

For the chosen skill, create `Skills/{name}/SKILL.md` in the vault. In Hebrew, following exactly the same conventions as every AIOS skill. The skeleton is ready to fill in, not empty:

```bash
NAME="lead-followup"   # replace with the chosen name
mkdir -p "Skills/$NAME"
```

Write `Skills/{name}/SKILL.md` in this structure:

```markdown
---
name: {name}
description: "{goal in one sentence} + the concrete behaviors + trigger phrases in Hebrew and English. Use when user says '{trigger}' or runs /{name}."
---

# {readable Hebrew title}

> [!note] {one line: what the skill does and for whom.}

## Step 1: {the first step}
{what happens here}

## Step 2: {the next step}
{...}

## Step 3: {the last step}
{...}

## Rules

1. {first hard rule}
2. {second hard rule}
3. Never use an em dash. Comma, period, or colon.
```

Fill the steps from the "Steps" column of the chosen proposal. If the skill needs to run on a schedule, add a `## Scheduling` section that explains how to schedule it in a cross-platform way (the Claude Desktop app, "Routines" / a local task, or the `schedule` skill; cron is a macOS-only option), and notes that the computer must be on at run time. If it touches an external system, add a ```bash block with the real command.

Explain that this is a starting skeleton: the steps are in broad strokes, and it is worth running it once and refining against reality.

## Step 5: Log and close

Add a log line to `System/logs/YYYY-MM-DD.md` and update `System/health/status.md` with the new skill.

```bash
DATE=$(date +%F); TIME=$(date +%H:%M); mkdir -p System/logs
printf -- '- %s | level-up | ✅ הצליח | נבנה שלד למיומנות %s | משך: %ss\n' \
  "$TIME" "$NAME" "$DUR" >> "System/logs/$DATE.md"
```

In the chat, return: the name and path of the new skill, and a sentence on the next step (run it once and refine). If there are more good opportunities left from the proposals, note that you can run `/level-up` again to build the next one.

## Rules

1. Always five questions before a proposal. Do not skip straight to building.
2. Concrete proposals only: name, trigger, goal in one sentence, steps. Not vague ideas.
3. Tie each proposal to the answer it came from. Without that it is generic.
4. Do not build before the user picks. Always `AskUserQuestion`.
5. The skeleton is written to `Skills/{name}/SKILL.md` only, following the AIOS SKILL.md conventions (frontmatter with two keys, Hebrew title, `## Step N`, `## Rules`).
6. The skeleton is ready to fill in, not empty. Real steps from the proposal.
7. Every run writes a log line and updates `System/health/status.md`.
8. Never use an em dash. Comma, period, colon, or split into sentences.
9. One skill per run. If they want more, run again.

## Troubleshooting

- **The answers are vague**: ask one specific follow-up question ("which action took you the most time this week?"). Do not invent a proposal out of thin air.
- **A similar skill already exists**: do not duplicate. Instead, offer to refine the existing one, or propose a different skill from the proposals.
- **The chosen skill is too big**: split it. Build the small working version first, and leave the extension for the next run of `/level-up`.
