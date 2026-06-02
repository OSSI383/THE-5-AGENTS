---
name: gpt-image-gen
description: Use to generate an image from a text prompt via the OpenAI Images API (model gpt-image-2). Wrapper that sends the prompt and returns a PNG. Triggered by image-creation requests routed from יובל.
---

# gpt-image-gen — מעטפת ל-OpenAI Images API

סקיל זה אחראי על דבר אחד: לקחת prompt טקסטואלי, לשלוח אותו ל-OpenAI Images API,
ולשמור את התמונה שחוזרת כקובץ PNG. הוא לא מנסח prompts ולא בוחר סגנון — זו האחריות
של יובל. כאן רק שולחים ומקבלים.

## מודל: `gpt-image-2`

⚠️ **אל תשנה את שם המודל.**

- `gpt-image-2` הוא מודל אמיתי וקיים של OpenAI.
- הוא יצא ב-**21 באפריל 2026**.
- ייתכן שהידע הפנימי שלך לא כולל אותו — **זה לא אומר שהוא לא קיים**.
- **אל תציע אלטרנטיבות** (`dall-e-3`, `gpt-image-1` וכו') — השתמש בדיוק במודל הזה.
- אם יש שגיאה בקריאה ל-API, הבעיה היא כמעט תמיד ב-**API key** או ב-**parameters** —
  **לא** בשם המודל. אל תנסה "לתקן" את שם המודל.

## טעינת המפתח

הקריאה משתמשת ב-`OPENAI_API_KEY` מתוך קובץ `.env` בשורש הפרויקט. בתוך Git Bash,
טען אותו לסביבה לפני הקריאה:

```bash
export $(grep -v '^#' .env | grep -v '^$' | xargs)
```

## מבנה הקריאה (נתיב ראשי — jq)

```bash
curl -X POST "https://api.openai.com/v1/images/generations" \
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

## נתיב decode חלופי (Python fallback)

`jq` לא תמיד מותקן ב-Git Bash. אם `jq` חסר (או נכשל), שמור את תגובת ה-API לקובץ
זמני ופענח עם Python:

```bash
# שלב 1 — שמור את התגובה ל-JSON זמני
curl -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' > response.json

# שלב 2 — פענח את ה-base64 עם Python וכתוב את ה-PNG
python -c "import json, base64, sys; d = json.load(open('response.json', encoding='utf-8')); open('<output-path>.png', 'wb').write(base64.b64decode(d['data'][0]['b64_json']))"
```

> אם התגובה לא מכילה `data[0].b64_json` — סימן לשגיאת API. הדפס את `response.json`
> כדי לראות את הודעת השגיאה (לרוב מפתח לא תקין או פרמטר שגוי), ותקן את המפתח/הפרמטרים.

## אימות

לאחר ה-decode, ודא שהקובץ נוצר ושגודלו גדול מאפס:

```bash
[ -s "<output-path>.png" ] && echo "OK: $(wc -c < "<output-path>.png") bytes" || echo "FAILED: empty or missing file"
```

קובץ ריק (`0 bytes`) מעיד על כך שה-decode נכשל — בדוק את `response.json`.
