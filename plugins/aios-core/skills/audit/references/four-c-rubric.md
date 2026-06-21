---
type: note
date: 2026-06-17
status: active
tags: [audit, rubric, 4c]
---

> [!note] How to score each of the four C's. For each C, the definition is given, plus what a score of 0, 50, and 100 looks like. Between those points, judge by your own assessment.

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

The four C's are the backbone of the audit and the upgrade. They measure how much of the business actually lives inside the system, and not just in the owner's head.

## Context

**What it is:** how much of the business is documented in `Context/` and the vault. This is the memory that every session and every routine reads first. Without context, the assistant works blind.

Measured against the 8 core files: `me.md`, `business.md`, `services.md`, `icp.md`, `brand.md`, `strategy.md`, `infrastructure.md`, `pain-points.md`. A file counts as filled only if it has real content, not a template with `[empty brackets]`.

| Score | What it looks like |
| --- | --- |
| 0 | No `Context/` folder, or every file is empty or template only. |
| 50 | About 4 of 8 files filled with real content. The base exists, but ICP, brand, or strategy are missing. |
| 100 | All 8 files filled, specific, and up to date. A routine can run without asking baseline questions. |

What drags the score down: files that are only a template, stale information (last year's strategy), or a generic `me.md` with no work style and no pain points.

## Connections

**What it is:** which external systems are actually connected and able to operate: Gmail, Drive, Calendar, WhatsApp, and additional data sources (CRM, analytics, ad account). A connection counts only with evidence: a "connected" status in `System/health/status.md`, or a recent successful run. An intention to connect is not a connection.

| Score | What it looks like |
| --- | --- |
| 0 | No system is connected. The system cannot read mail, calendar, or messages. |
| 50 | One or two connections work (for example Gmail), but central sources are missing (no WhatsApp, no business data source). |
| 100 | Every system the business needs is connected and active, with evidence of a recent successful run. |

What drags the score down: a connection that shows as "configured" but failed on its last run, or an expired token.

## Capabilities

**What it is:** which skills (`Skills/`) and subagents (`.claude/agents/`) exist and are healthy. These are the actions the system can perform beyond conversation: a morning report, mail triage, preparing a price proposal, and so on. A skill counts only with a valid `SKILL.md`.

| Score | What it looks like |
| --- | --- |
| 0 | No customized skills. The system only answers in chat, it does not act. |
| 50 | There are 1 to 2 skills that work, but central recurring actions are still done manually. |
| 100 | A skill for every meaningful recurring process in the business, each with a clear `SKILL.md` and trigger. |

What drags the score down: a skill with no `SKILL.md`, a skill that duplicates another, or a skill that has never run.

## Cadence

**What it is:** which routines run on their own on a schedule, and when they last ran. This is the difference between tools that wait to be called, and a system that pushes results. Measured by `System/logs/` and `System/health/status.md`. A routine that has not run for a week counts as stuck.

| Score | What it looks like |
| --- | --- |
| 0 | Nothing runs on its own. Every action requires someone to start it. |
| 50 | One routine is scheduled and running (for example a morning report), but it is alone and the rest is manual. |
| 100 | Several routines run on a schedule, all of them ran recently and successfully, and there is supervision that alerts when something breaks. |

What drags the score down: a scheduled routine that has not run for a week (stuck), or a routine with no run record (no way to know if it worked).

## A note on scheduling

Cadence depends on the computer being on at the right time. The design is to push results during the owner's working hours, not a 24/7 server. A routine that is "scheduled" but the computer is off at its time will not run. That is a real consideration in the cadence score.
