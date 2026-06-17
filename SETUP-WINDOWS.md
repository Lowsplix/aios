# התקנה על Windows (מדריך ללקוח)

> [!note] רשימת התקנה פשוטה למחשב Windows. עוברים עליה לפי הסדר, בדרך כלל יחד בשיתוף מסך. אין צורך בידע טכני: כל שלב הוא הורדה אחת או הדבקה אחת של שורה.

המערכת רצה על Windows באופן מקורי. צריך להתקין כמה דברים פעם אחת, ואז הכל עובד. עוברים על השלבים אחד-אחד, ולא ממשיכים לשלב הבא לפני שהקודם הסתיים. אם משהו נתקע, עוצרים ושואלים, אין לחץ.

לפני שמתחילים: כמה התקנות מבקשות הרשאת מנהל (admin). זה נורמלי. אם יקפוץ חלון של Windows ששואל "לאפשר לאפליקציה לבצע שינויים?", לוחצים **כן** (Yes).

## שלב 1: התקנת Claude Code

זה המוח של המערכת. מתקינים אותו דרך חלון בשם PowerShell.

1. לוחצים על כפתור התחל (Start), מקלידים `PowerShell`, ופותחים את **Windows PowerShell**.
2. מדביקים את השורה הבאה (קליק ימני מדביק ב-PowerShell), ולוחצים Enter:

```powershell
irm https://claude.ai/install.ps1 | iex
```

3. מחכים שההתקנה תסתיים. בסוף סוגרים את החלון.

> הערה: ההתקנה עשויה לבקש הרשאת מנהל. אם קופץ חלון אישור, לוחצים כן.

## שלב 2: התקנת Git for Windows (זה מה שמפעיל את הפקודות של המערכת)

זה החלק הכי חשוב בהתקנה. בלי זה, חלק מהפקודות של המערכת פשוט לא יעבדו. עם זה, הכל עובד חלק.

1. נכנסים לכתובת: `https://git-scm.com/download/win`
2. ההורדה מתחילה לבד (אם לא, לוחצים על הקישור המתאים ל-64-bit).
3. פותחים את הקובץ שירד ולוחצים **Next** בכל המסכים. **משאירים את כל ברירות המחדל כמו שהן**, לא צריך לשנות כלום. בסוף לוחצים **Install** ואז **Finish**.

> הערה: ההתקנה מבקשת הרשאת מנהל. לוחצים כן. הסיבה שמתקינים את זה: היא מביאה כלי בשם Git Bash, וזה מה שמאפשר למערכת להריץ את הפקודות שלה בצורה תקינה.

## שלב 3: התקנת Node.js (נחוץ לחיבור לגוגל)

זה מה שמאפשר למערכת להתחבר לג'ימייל, לדרייב וליומן שלך, דרך כלי בשם gws.

1. נכנסים לכתובת: `https://nodejs.org`
2. לוחצים על הכפתור הגדול שמסומן **LTS** (זו הגרסה היציבה).
3. פותחים את הקובץ שירד, לוחצים **Next** בכל המסכים, ומשאירים את ברירות המחדל. בסוף **Install** ואז **Finish**.

> הערה: ההתקנה מבקשת הרשאת מנהל. לוחצים כן.

## שלב 4: אפליקציית Claude Desktop (לא חובה, מומלץ לתזמון)

זה לא חובה, אבל זה מה שמאפשר למערכת לרוץ לבד בזמנים קבועים (למשל דוח בוקר כל יום ב-07:00) דרך "Routines" (משימות מקומיות).

1. נכנסים לכתובת: `https://claude.com/download`
2. מורידים את האפליקציה ל-Windows, פותחים את הקובץ, ומתקינים.
3. נכנסים עם אותו חשבון.

> שים לב: תזמון כזה רץ על המחשב שלך, אז המחשב צריך להיות דלוק בשעה שבה המשימה אמורה לרוץ. זו לא מערכת שרצה בענן 24/7, זו דחיפה של תוצאה בשעות העבודה שלך.

## שלב 5: הפעלת המערכת

עכשיו מתקינים את המערכת עצמה ומתחילים. פותחים חלון Claude Code (מאותו כפתור התחל, מקלידים `claude`), ומריצים את הפקודות האלה אחת-אחת:

1. הוספת המאגר של אדיר:

```
/plugin marketplace add <כתובת המאגר>
```

2. התקנת המערכת:

```
/plugin install aios-core@aios
```

3. פותחים תיקייה ריקה חדשה (זו תהיה "הכספת", הבית של המערכת), ובתוכה מריצים:

```
/onboard
```

זה מקים את השלד ושואל אותך כמה שאלות קצרות כדי למלא את המערכת בך ובעסק שלך.

4. אחרי ההקמה, מחברים את המערכת לעולם (גוגל, ועוד):

```
/connect
```

זהו. מכאן המערכת מוכנה. בכל רגע אפשר להריץ `/doctor` כדי לראות שהכל מחובר ועובד.

## מה לא עובד עדיין על Windows (וזה בסדר)

ממשק הוואטסאפ (לשלוח ולקרוא הודעות דרך המערכת) רץ כרגע רק על מק (macOS), כי הוא מבוסס על כלי שלא קיים ל-Windows. זה לא חוסם כלום: המערכת פשוט כותבת את הדוחות והתשובות לכספת במקום לשלוח בוואטסאפ. חיבור הוואטסאפ דורש מכונת מק או ממסר (relay), והוא מתוכנן כתוספת קרובה.

---

# Windows Setup (client guide, English mirror)

Plain checklist for installing on a Windows PC, in order. No technical knowledge needed. Some steps need admin rights (click Yes if Windows asks to allow changes).

1. **Claude Code**: open Windows PowerShell, paste `irm https://claude.ai/install.ps1 | iex`, press Enter. (May need admin.)
2. **Git for Windows** (the most important step, this is what makes the system's commands work): go to `https://git-scm.com/download/win`, run the installer, click Next on every screen, keep all defaults, Install, Finish. (Needs admin. It installs Git Bash, which the system's commands run inside.)
3. **Node.js LTS** (needed for the Google connection via gws): go to `https://nodejs.org`, click the **LTS** button, run the installer, keep defaults. (Needs admin.)
4. **Claude Desktop app** (optional, recommended for scheduling): go to `https://claude.com/download`, install, sign in. Used for "Routines" (local tasks). The PC must be on at the scheduled time, this is not a 24/7 cloud server.
5. **Start the system**: open Claude Code, then run, one at a time: `/plugin marketplace add <repo>`, then `/plugin install aios-core@aios`, then open an empty folder (your future vault) and run `/onboard`, then `/connect`. Run `/doctor` anytime to check everything is connected.

**Not available on Windows yet (that's fine):** the WhatsApp interface runs on macOS only for now (its underlying tool has no Windows build). Nothing breaks: the system writes reports and answers to the vault instead of sending them on WhatsApp. WhatsApp needs a macOS machine or a relay, planned as a fast-follow.
