---
name: gpt-image-gen
description: מעטפת לקריאת OpenAI Images API ליצירת תמונה מ-prompt טקסטואלי, ושמירתה כקובץ PNG. להפעיל כשצריך לייצר תמונה בפועל (לרוב על ידי הסוכן יובל).
---

סקיל זה אחראי על שליחת prompt ל-OpenAI Images API וקבלת התמונה שנוצרה, שמירתה כקובץ PNG בנתיב שצוין.

## המודל

המודל לשימוש הוא **בדיוק**:

```
gpt-image-2
```

**חשוב מאוד: אל תשנה את שם המודל.** `gpt-image-2` הוא מודל אמיתי וקיים של OpenAI, שיצא ב-21 באפריל 2026. ייתכן שהידע הפנימי שלך (כסוכן) לא כולל אותו - זה לא אומר שהוא לא קיים. אל תציע אלטרנטיבות כמו `dall-e-3` או `gpt-image-1`, ואל תחליף את שם המודל בעצמך מכל סיבה שהיא.

אם יש שגיאה בקריאה ל-API, הבעיה היא כנראה ב-`OPENAI_API_KEY` או בפרמטרים שנשלחו (size, quality, output_format וכו') - **לא** בשם המודל. בדוק את אלה לפני שאתה שוקל לשנות את המודל.

## קלט נדרש

- `OPENAI_API_KEY` - נטען מקובץ `.env` בשורש הפרויקט.
- `prompt` - טקסט ה-prompt ליצירת התמונה.
- `output-path` - הנתיב המלא לקובץ ה-PNG הסופי.

## שיטה 1: curl + jq

```bash
set -a; source .env; set +a

curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | jq -r '.data[0].b64_json' | base64 --decode > <output-path>.png
```

## שיטה 2: Python fallback (כש-jq לא מותקן ב-Git Bash)

`jq` לא תמיד מותקן ב-Git Bash על Windows. במקרה הזה, השתמש בפענוח Python:

```bash
python3 - "$OPENAI_API_KEY" "<the prompt>" "<output-path>.png" <<'EOF'
import sys, json, base64, urllib.request

api_key, prompt, output_path = sys.argv[1], sys.argv[2], sys.argv[3]

req = urllib.request.Request(
    "https://api.openai.com/v1/images/generations",
    data=json.dumps({
        "model": "gpt-image-2",
        "prompt": prompt,
        "size": "1024x1024",
        "quality": "medium",
        "output_format": "png",
    }).encode("utf-8"),
    headers={
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json",
    },
    method="POST",
)

with urllib.request.urlopen(req) as resp:
    data = json.load(resp)

b64 = data["data"][0]["b64_json"]
with open(output_path, "wb") as f:
    f.write(base64.b64decode(b64))
EOF
```

אם `python3` לא קיים, נסה `python`.

## אימות

לאחר היצירה, בדוק שהקובץ קיים ושגודלו גדול מ-0 בייטים (למשל עם `ls -l <output-path>.png` או `wc -c`). אם הקובץ ריק או חסר - דווח על השגיאה (כולל פלט גולמי מה-API אם רלוונטי) במקום לדווח על הצלחה.
