---
name: update
description: מושך את הגרסה האחרונה של תוסף ה-AIOS מהמאגר המרכזי של אדיר, כך שכל המיומנויות של הלקוח מתעדכנות בלי שהלקוח צריך לעשות כלום. רץ ידנית או בתזמון יומי לעדכון אוטומטי. Use when user says "update", "עדכון", "עדכן את המערכת", or runs /update. Maintainer-facing self-update for the aios-core plugin.
---

# עדכון המערכת

> [!note] מושך את הגרסה האחרונה של התוסף מהמאגר של אדיר ומעדכן את כל המיומנויות. הלקוח לא צריך לבנות כלום.

המיומנויות חיות בתוסף Claude Code אחד (`aios-core`) שמותקן מהמאגר המרכזי של [[אדיר סלם]]. כשאדיר משפר מיומנות אצל לקוח אחד, הוא דוחף את השינוי למאגר, וכל לקוח מושך אותו. ככה כולם נשארים מעודכנים מנקודה אחת.

## שלב 1: איתור תיקיית התוסף

תיקיית המאגר נקבעת בהתקנה ונשמרת ב-`System/health/status.md` בשדה `plugin_dir`. אם אין, חפש את המאגר המשוכפל (clone) של `aios`.

```bash
PLUGIN_DIR="$(grep -m1 'plugin_dir:' System/health/status.md 2>/dev/null | sed 's/.*plugin_dir: *//')"
[ -z "$PLUGIN_DIR" ] && PLUGIN_DIR="$(find "$HOME" -maxdepth 4 -type d -name aios -path '*aios*' 2>/dev/null | head -1)"
echo "תיקיית התוסף: $PLUGIN_DIR"
OLD_VER="$(python3 -c "import json;print(json.load(open('$PLUGIN_DIR/plugins/aios-core/.claude-plugin/plugin.json'))['version'])" 2>/dev/null)"
```

## שלב 2: משיכת העדכון

```bash
git -C "$PLUGIN_DIR" pull --ff-only
NEW_VER="$(python3 -c "import json;print(json.load(open('$PLUGIN_DIR/plugins/aios-core/.claude-plugin/plugin.json'))['version'])" 2>/dev/null)"
echo "גרסה: $OLD_VER -> $NEW_VER"
```

אם זמין, רענן גם את רישום התוסף ב-Claude Code (`/plugin marketplace update` בסשן אינטראקטיבי). שינויים במיומנויות נטענים בסשן הבא.

## שלב 3: רישום ודיווח

עדכן את `System/health/status.md` (שורת `aios-core version` + חותמת זמן) והוסף שורת לוג ל-`System/logs/YYYY-MM-DD.md` בפורמט הסטנדרטי:

```
- HH:MM | update | ✅ הצליח | עודכן מ-OLD ל-NEW | משך: Ns
```

דווח ללקוח במשפט אחד בעברית: אם לא היה עדכון, "המערכת כבר מעודכנת (גרסה X)". אם היה, "עודכנת לגרסה Y, השינויים ייכנסו לתוקף בסשן הבא".

## תזמון

להפעלה אוטומטית, תזמן את `/update` פעם ביום (למשל 06:00) דרך מיומנות ה-`schedule` או cron/launchd. ככה הלקוח מתעדכן לבד אחרי שאדיר דוחף שיפור. שים לב: המחשב צריך להיות דלוק בזמן התזמון.

## חוקים

- משיכה בלבד (`--ff-only`). לעולם אל תשנה או תדחוף קוד מצד הלקוח. הלקוח צרכן, לא עורך.
- אם ה-pull נכשל (קונפליקט, אין רשת), אל תיגע בכלום, רשום `❌ נכשל` עם הסיבה, ודווח לאדיר.
- תמיד רשום גרסה לפני ואחרי, כדי שאפשר יהיה לדעת מי על איזו גרסה.
