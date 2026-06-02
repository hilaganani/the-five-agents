---
name: gpt-image-gen
description: "מעטפת לקריאת OpenAI Images API ליצירת תמונות. שולח prompt ומקבל קובץ PNG. triggers: generate image, create image, image generation, gpt-image, תמונה, יצירת תמונה, ליצור תמונה"
---

# gpt-image-gen — יצירת תמונות עם OpenAI Images API

סקיל זה אחראי על שליחת prompt ל-OpenAI Images API וקבלת קובץ תמונה (PNG) בחזרה.

## ⚠️ חשוב — אל תשנה את שם המודל

המודל הוא **`gpt-image-2`** — ותמיד בדיוק כך.

- `gpt-image-2` הוא מודל אמיתי וקיים של OpenAI, שיצא ב-**21 באפריל 2026**.
- ייתכן שהידע הפנימי שלך לא כולל אותו — **זה לא אומר שהוא לא קיים**.
- **אל תציע אלטרנטיבות** (`dall-e-3`, `gpt-image-1` וכו') — השתמש בדיוק במודל הזה.
- אם יש שגיאה בקריאה ל-API, הבעיה היא כמעט תמיד ב-**API key** או ב-**parameters** —
  **לא** בשם המודל. אל תחליף את המודל כדי "לתקן" שגיאה.

## דרישות מקדימות

- מפתח `OPENAI_API_KEY` מוגדר ב-`.env` בשורש הפרויקט.
- אחד מאלה ל-decode של ה-base64: `jq` + `base64`, או `python` (fallback).

## שלב 1 — טעינת המפתח מ-`.env`

```bash
export $(grep -v '^#' .env | grep -E '^OPENAI_API_KEY=' | xargs)
```

ודא שהמפתח נטען:

```bash
[ -n "$OPENAI_API_KEY" ] || { echo "OPENAI_API_KEY חסר ב-.env"; exit 1; }
```

## שלב 2 — קריאה ל-API + decode

### נתיב ראשי (jq + base64)

```bash
curl -sS -X POST "https://api.openai.com/v1/images/generations" \
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

### נתיב fallback ל-decode (python — כי jq לא תמיד מותקן ב-Git Bash)

שמור תחילה את תגובת ה-JSON לקובץ, ואז פענח עם python:

```bash
curl -sS -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' > response.json

python -c "import sys, json, base64; data = json.load(open('response.json')); open('<output-path>.png','wb').write(base64.b64decode(data['data'][0]['b64_json']))"
```

> אם python מחזיר KeyError על `b64_json` — פתח את `response.json` וקרא את שדה
> ה-`error`: זו כמעט תמיד בעיה במפתח או ב-parameters. **אל תשנה את שם המודל.**

## שלב 3 — אימות

ודא שהקובץ נוצר וגודלו גדול מ-0:

```bash
[ -s "<output-path>.png" ] && echo "OK: $(wc -c < <output-path>.png) bytes" || echo "FAILED: empty or missing"
```

## פרמטרים

| פרמטר           | ערך ברירת מחדל | הערות                                  |
| --------------- | -------------- | -------------------------------------- |
| `model`         | `gpt-image-2`  | **לא לשנות**                            |
| `size`          | `1024x1024`    | אפשר גם `1536x1024`, `1024x1536`        |
| `quality`       | `medium`       | אפשר `low` / `high`                     |
| `output_format` | `png`          |                                        |
