# Morning report template

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

This is the deliverable the client reads each morning, so it is rendered to the user in Hebrew. The structure below is the contract: keep the frontmatter, the five section headings, and the checklist for tasks exactly as shown. The Hebrew section names (`## הזדמנויות חדשות`, `## החלטות שצריך לקבל`, `## משימות להיום`, `## לידים`, `## בריאות המערכת`) are the headings the client sees, so they stay in Hebrew verbatim. Fill each `[bracketed]` hint with real content in Hebrew, replace `YYYY-MM-DD` with today's date, and weave `[[wikilinks]]` for every entity. When a section has nothing to report, use the Hebrew "none" fallback shown in its hint. The fenced block below is the exact template to render.

```markdown
---
type: report
date: YYYY-MM-DD
status: active
tags: [morning-report, daily]
---

> [!note] דוח בוקר ל-YYYY-MM-DD. תמונה אחת לפתוח איתה את היום.

## הזדמנויות חדשות

- [דבר חדש ששווה לקפוץ עליו, עם [[קישור]] לישות. אם אין: "אין הזדמנויות חדשות הבוקר".]

## החלטות שצריך לקבל

- [החלטה שממתינה לך וחוסמת התקדמות, עם [[קישור]] לפרויקט. אם אין: "אין החלטות פתוחות".]

## משימות להיום

- [ ] [המשימה הכי חשובה היום, עם דדליין אם יש]
- [ ] [משימה דחופה נוספת]
- [ ] [משימה שלישית]

## לידים

- [פנייה חדשה שלא טופלה, עם מקור ו-[[קישור]] אם ידוע. אם אין: "אין לידים חדשים".]

## בריאות המערכת

[שורה אחת: מה רץ, מה נכשל, מה מחובר. למשל: "הכל רץ. וואטסאפ מחובר, דוח בוקר אתמול 07:00 ✅." או: "⚠️ הסנכרון של וואטסאפ לא רץ מאז אתמול, כדאי לבדוק."]
```
