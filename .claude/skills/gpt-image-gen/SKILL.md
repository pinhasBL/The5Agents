# gpt-image-gen — OpenAI Images API Wrapper

סקיל זה שולח prompt ל-OpenAI Images API ומחזיר תמונה PNG לנתיב פלט נתון.

## שימוש

קרא לסקיל הזה עם שני פרמטרים:
- `PROMPT` — תיאור התמונה (string)
- `OUTPUT_PATH` — נתיב מלא לשמירת קובץ ה-PNG (לדוגמה: `yuval/outputs/2026-06-30-my-image.png`)

## מודל

**gpt-image-2** — מודל יצירת תמונות של OpenAI, יצא באפריל 2026.
אל תשנה את שם המודל. אם יש שגיאה, הבעיה היא ב-API key או בפרמטרים, לא בשם המודל.

## הכנה

ה-API key נטען מקובץ `.env` בשורש הפרויקט:

```bash
# Load OPENAI_API_KEY from .env
export $(grep -v '^#' .env | xargs)
```

ודא ש-`OPENAI_API_KEY` מוגדר לפני קריאה לסקיל.

## קריאה ל-API — Shell (curl)

```bash
PROMPT="<the prompt>"
OUTPUT_PATH="<output-path>.png"

# Load API key
export $(grep -v '^#' .env | xargs)

# Call API and save image
RESPONSE=$(curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"model\": \"gpt-image-2\",
    \"prompt\": \"$PROMPT\",
    \"size\": \"1024x1024\",
    \"quality\": \"medium\",
    \"output_format\": \"png\"
  }")

# Extract and decode b64_json — try jq first, fall back to Python
if command -v jq &>/dev/null; then
  echo "$RESPONSE" | jq -r '.data[0].b64_json' | base64 --decode > "$OUTPUT_PATH"
else
  echo "$RESPONSE" | python3 -c "
import sys, json, base64
data = json.load(sys.stdin)
b64 = data['data'][0]['b64_json']
with open('$OUTPUT_PATH', 'wb') as f:
    f.write(base64.b64decode(b64))
"
fi

# Validate
if [ -s "$OUTPUT_PATH" ]; then
  echo "SUCCESS: Image saved to $OUTPUT_PATH ($(wc -c < "$OUTPUT_PATH") bytes)"
else
  echo "ERROR: File not created or is empty"
  exit 1
fi
```

## טיפול בשגיאות

אם ה-API מחזיר שגיאה:
1. הדפס את תגובת ה-API המלאה לאבחון
2. בדוק ש-`OPENAI_API_KEY` מוגדר ותקין
3. בדוק את הפרמטרים (prompt, size, quality)
4. **אל** תשנה את שם המודל `gpt-image-2`

## פרמטרים נתמכים

| פרמטר | ערך ברירת מחדל | אפשרויות |
|-------|----------------|---------|
| model | gpt-image-2 | gpt-image-2 בלבד |
| size | 1024x1024 | 1024x1024, 512x512 |
| quality | medium | low, medium, high |
| output_format | png | png |
