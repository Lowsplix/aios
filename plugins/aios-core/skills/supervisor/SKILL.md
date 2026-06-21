---
name: supervisor
description: "The WhatsApp control interface for the AIOS vault. The owner messages questions or commands over WhatsApp; this skill runs wacli doctor first as a liveness gate (aborts with a clear Hebrew message if sync is stalled), reads the owner's recent messages via wacli, interprets each as a query or command against the vault (answers questions from Context/Projects/Tasks, surfaces tasks, runs a report), and replies over WhatsApp via wacli send. Acts only on messages the owner initiated, never sends without a clear answer, and logs every action to System/logs/. Use when the user says 'מפקח', 'תבדוק וואטסאפ', 'מה שאלו בוואטסאפ', 'תענה בוואטסאפ', 'supervisor', 'whatsapp control', or runs /supervisor."
---

# Supervisor (WhatsApp control)

> [!note] The system's control interface over WhatsApp. You send a question or a command from your phone, the supervisor reads it, understands what you asked, looks up the answer in the vault, and replies to you over WhatsApp. Everything is logged.

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

The idea: you are not always at the computer, but the phone is always on you. The supervisor lets you operate the vault from a WhatsApp message. "What are my tasks today?", "What is the status of project X?", "Run the morning report". It understands, acts against the vault, and replies.

It is important to understand what the supervisor is **not**: it is not a bot that talks to customers, it does not initiate messages, and it does not reply to everyone who writes. It replies only to you, the owner, and only to messages you initiated.

This is steps. Run them in order. Speak Hebrew, short and clear.

## Step 0: WhatsApp availability gate (is wacli installed)

The supervisor's WhatsApp channel runs on `wacli`. The tool works on every operating system (Mac, Windows, Linux), but it may not be installed and connected yet. The gate here checks whether `wacli` is even **installed**, not which OS you are on. If it is not installed, that is fine, do not fail over it.

```bash
if ! command -v wacli >/dev/null 2>&1; then
  echo "wacli is not installed yet. Working in WhatsApp-disconnected mode: answering in the session and writing to the vault."
fi
```

If `wacli` is not installed, work in **WhatsApp-disconnected mode** and do not stop:
- Do not try to read or send WhatsApp messages. There is no message source.
- If you are in an interactive session and the user asked you something directly, treat their question as an "owner message": interpret it per step 3, answer in the session, and where appropriate write the answer as a note to the vault (for example to `Daily/YYYY-MM-DD.md` or the relevant project) instead of sending over WhatsApp.
- Write a `⚠️ חלקי` log line with the line "WhatsApp לא זמין: wacli לא מותקן", and answer in the session.
- Note briefly: the WhatsApp interface requires installing and connecting `wacli` (run `/connect` and choose WhatsApp). Skip steps 1, 2, and 4 (all WhatsApp-dependent) and go straight to step 5 (logging).

Only if `wacli` is installed, continue to step 1 (where we check that it is also alive and synced).

## Step 1: Liveness gate (mandatory, first)

**Always check liveness first.** The WhatsApp sync can stall silently: the process looks alive but the WebSocket is dead and it is not receiving new messages. If you skip the check, you might read old messages, miss new ones, or send a reply that goes into the void.

> [!warning] Do not trust the `connected` field of `wacli doctor`
> While the sync process is running (which is the healthy state, almost always), it holds the store lock, and `wacli doctor` returns `connected: false` and `connection_state: locked_by_other_process` regardless of the real state. If you stop on `connected: false`, you will mistakenly think WhatsApp is disconnected exactly when it is working. The reliable liveness signal is `last_sync_at`: when a sync actually last came in. (The full always-on connection mechanism is in `wacli-always-on.md` in the `connect` skill.)

```bash
LAST_SYNC=$(wacli --read-only --json doctor 2>/dev/null \
  | python3 -c 'import sys,json;print(json.load(sys.stdin).get("data",{}).get("store",{}).get("last_sync_at",""))')
echo "סנכרון אחרון: ${LAST_SYNC:-לא ידוע}"
```

If `last_sync_at` is empty, or much older than the reasonable time window (for example, it has not updated for several hours during active hours), the sync is stalled:

> [!warning] Stop. Do not read messages and do not send. Write a log line with ❌ נכשל, and note that WhatsApp is not connected. If you are in an interactive session, tell the user in Hebrew: "הוואטסאפ לא מסונכרן כרגע (`wacli doctor` מראה תקיעה). לא קראתי ולא עניתי. כדאי לבדוק את החיבור ולהריץ שוב."

Only if `wacli doctor` is healthy, continue to step 2.

## Step 2: Read the owner's messages

Read the latest messages from the owner only. The owner's number/id is saved in `Context/me.md` or `Context/infrastructure.md`, read it from there.

```bash
# replace OWNER with the owner's number/id from the context
# list the latest messages from the owner (adapt the flags to the installed wacli version)
wacli messages --from "$OWNER" --limit 10
```

Rules on reading:
- Read only messages from the owner. Ignore every other conversation.
- Handle only messages **the owner initiated** that have not been handled yet (new messages since the previous run). Do not reply again to a message you already answered.
- To know what has already been handled, keep a list of answered message ids in `System/supervisor-seen.md` (create if it does not exist). Skip what is already on the list.
- If there are no new messages from the owner: there is nothing to do. Write a short log line ("no new messages") and finish.

## Step 3: Interpret each message

For each new message from the owner, understand whether it is a **question** (needs an answer from the vault) or a **command** (needs an action performed). Map by intent:

| Message type | Example | What to do |
| --- | --- | --- |
| Question about status | "What are my tasks today?" | Read `task-list/Tasks.md`, filter to the relevant, answer. |
| Question about a project | "What is happening with [[project]]?" | Read `Projects/{name}/`, answer with the status and the next step. |
| Question about info | "What is the phone number of supplier X?" | Search `Context/` and `Intelligence/` with `grep`, answer. |
| Report command | "Run the morning report", "Send me a summary" | Run `/morning-report`, and send the brief back. |
| Logging command | "Add a task: call X" | Add to `task-list/Tasks.md` in task-board format, confirm briefly. |
| Unclear | A vague or partial message | Do not guess. Ask for a short clarification over WhatsApp ("not sure I understood, did you mean...?"). |

To search the vault, scan, do not read whole files. Use `grep` and `ls`:

```bash
grep -rni "<the keyword from the message>" Context/ Projects/ Intelligence/ task-list/ 2>/dev/null | head -20
```

## Step 4: Reply over WhatsApp

For every handled message, send one clear reply to the owner.

```bash
# replace OWNER with the target, and REPLY with the answer
wacli send --to "$OWNER" --text "$REPLY"
```

Rules on replying:
- **Never send without a clear answer.** If you did not find the info, that is a valid answer: "I did not find it in the vault. Want me to add it?" but never send a guess, a half-answer, or "I am checking" without a follow-up.
- The reply is in clean text, without Obsidian syntax (no `[[ ]]`, no `> [!note]`, no `#tags`). WhatsApp shows plain text.
- Short and to the point. If the answer is long (for example a full morning report), send it as one well-ordered message in lines.
- After a successful send, add the message id to `System/supervisor-seen.md` so it is not answered again.

## Step 5: Run-log and health

Every action is logged. Write a log line per run, and note how many messages were handled and what was done.

```bash
DATE=$(date +%F); TIME=$(date +%H:%M); mkdir -p System/logs System/health
printf -- '- %s | supervisor | ✅ הצליח | %s הודעות טופלו, %s תשובות נשלחו | משך: %ss\n' \
  "$TIME" "$HANDLED" "$SENT" "$DUR" >> "System/logs/$DATE.md"
```

(`$HANDLED` = how many messages were processed, `$SENT` = how many replies were sent, `$DUR` = duration in seconds. If `wacli doctor` failed and therefore nothing was read, log ❌ נכשל with the reason "וואטסאפ לא מסונכרן". If some messages were answered and some were not, log ⚠️ חלקי.)

Update the `supervisor` line in `System/health/status.md`: last timestamp, last status (including whether WhatsApp was connected), and how many messages were handled.

## Step 6: Summary in the chat

If running in an interactive session, return a short summary: how many owner messages were handled, what was asked, and what you answered. If WhatsApp was not connected, say that first.

## Scheduling

You can run the supervisor manually whenever you want it to check what is waiting for you on WhatsApp, or schedule it to run every few minutes during working hours so it answers you almost immediately.

> [!important] The computer must be on and open for the supervisor to read and reply. This is a push of a result during your working hours, not a 24/7 server. When the computer is off, messages you send simply wait until the next run when the computer comes back up.

To schedule (works on every operating system): use the Claude Desktop app, "Routines" (a local task), or run the `schedule` skill, and ask it to schedule `/supervisor` at a frequency that suits you during working hours (for example every 10 minutes between 08:00 and 19:00). The task runs on this computer, so it must be on. Remember: every run opens with a check of whether `wacli` is installed and then with `wacli doctor`. If `wacli` has not been installed and connected yet, the supervisor works in WhatsApp-disconnected mode: answering in the session and writing to the vault. When `wacli` is present, even if the sync stalls between runs, the next run will detect it and not act on old info.

## Rules

1. **Availability gate first (is `wacli` installed), then `wacli doctor` first.** If `wacli` is not installed, work in WhatsApp-disconnected mode: answer in the session and write to the vault, log ⚠️ חלקי, do not fail. If `wacli` is installed but the sync is stalled, stop: do not read and do not send, log ❌ נכשל and say WhatsApp is not connected. The distinction is by the actual `wacli` state, not by the operating system.
2. Act only on messages **the owner initiated** themselves. Ignore every other conversation and every other sender.
3. Do not answer the same message twice. Keep a list of answered ids in `System/supervisor-seen.md`.
4. **Never send without a clear answer.** "I did not find it, want me to add it?" is an answer. A guess or "I am checking" without a follow-up is not.
5. An unclear message: ask for a short clarification, do not guess the intent.
6. WhatsApp replies in clean text, without Obsidian syntax.
7. Every action is logged: a log line to `System/logs/` and an update to `System/health/status.md`.
8. To search the vault, scan with `grep` and `ls`. Do not read whole files.
9. Never use an em dash. Comma, period, colon, or split into sentences.
10. The supervisor answers the owner only. It is not a customer interface and does not initiate messages to anyone.

## Troubleshooting

- **`wacli` is not installed**: this is fine, not a fault. Work in WhatsApp-disconnected mode: answer in the session and write to the vault, log ⚠️ חלקי with "WhatsApp לא זמין: wacli לא מותקן". To connect WhatsApp, run `/connect` and choose WhatsApp. The tool works on Mac, Windows, and Linux.
- **`wacli doctor` shows a stall or disconnection**: this is the common scenario. Stop immediately, do not act on old info. Log ❌ נכשל, and in `System/health/status.md` mark that WhatsApp is not connected. If interactive, tell the user to check the connection and run again.
- **No new messages from the owner**: not an error. Write a short log line ("no new messages") and finish without sending anything.
- **A message asks for something forbidden or impossible** (for example, sending a message to a customer): do not do it. Reply to the owner briefly that it is out of the supervisor's scope.
- **The send failed after the answer is ready**: log ⚠️ חלקי with the reason, and do not add the message to `supervisor-seen.md` so it is retried on the next run.
- **The owner's number is not found in the context**: stop. Without knowing who the owner is, you cannot safely filter messages. Say that the owner's number needs to be filled in `Context/me.md`.
