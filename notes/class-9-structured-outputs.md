# Class 9: Structured Outputs

## What You'll Learn

- What structured outputs are and why they exist
- The problem with parsing free-form AI text
- How to get the AI to return JSON with a specific schema
- How Gemini's `responseMimeType` and `responseSchema` work
- Eliminating the `cleanGeneratedCode` regex function
- Getting the AI to generate titles automatically
- Updating TypeScript types for the new response format

## What You'll Build

By the end of this class, your AI Component Builder will use structured outputs to get clean, predictable JSON from Gemini - with both a `title` and `code` field. No more regex cleaning. No more manually extracting titles.

---

## The Problem: Free-Form Text is Unpredictable

Right now, we ask the AI to "return only raw JSX" and then run a function called `cleanGeneratedCode` to strip out markdown fences, imports, exports, and function wrappers:

```typescript
const cleanGeneratedCode = (raw: string): string => {
  let code = raw.trim();
  code = code.replace(/^```(?:jsx|tsx|javascript|typescript)?\s*\n?/i, '');
  code = code.replace(/\n?```\s*$/i, '');
  code = code.replace(/^import\s+.*;\s*\n?/gm, '');
  code = code.replace(/^export\s+(default\s+)?/gm, '');
  const fnMatch = code.match(/(?:function|const)\s+\w+\s*(?:=\s*)?...$/);
  if (fnMatch?.[1]) {
    code = fnMatch[1].trim();
  }
  return code.trim();
};
```

This works most of the time, but it has problems:

1. **Fragile**: If the AI returns a slightly different format, the regex might miss something
2. **Incomplete**: The regex can't catch every possible wrapper format
3. **Maintenance burden**: You're writing parser code for something the AI should handle
4. **Title extraction is hacky**: We just take the first 6 words of the prompt as a title

What if we could tell the AI: "Give me a JSON object with exactly two keys: `title` and `code`"?

---

## The Solution: Structured Outputs

Structured outputs tell the AI model to return data in a specific format (usually JSON) that matches a schema you define. Instead of getting raw text that you have to parse, you get structured data that you can use directly.

### Without Structured Outputs

```
You: "Make me a dark pricing card"
AI:  "```jsx\n<div className=\"bg-gray-900\">...</div>\n```"
You: *runs 5 regex passes to clean the response*
```

### With Structured Outputs

```
You: "Make me a dark pricing card" + schema: { title: string, code: string }
AI:  { "title": "Dark Pricing Card", "code": "<div className=\"bg-gray-900\">...</div>" }
You: *directly uses response.title and response.code*
```

No regex. No parsing. No edge cases.

---

## How It Works in Gemini

Gemini supports structured outputs through two configuration options:

1. **`responseMimeType: "application/json"`** - tells Gemini to return valid JSON
2. **`responseSchema`** - defines the exact shape of the JSON

Here's the updated `handleGenerate` function:

```typescript
import { GoogleGenerativeAI, SchemaType } from '@google/generative-ai';

const handleGenerate = useCallback(async (prompt: string) => {
  if (!apiKey) return;
  setGenerationState({ status: 'loading' });

  try {
    const genAI = new GoogleGenerativeAI(apiKey);
    const model = genAI.getGenerativeModel({
      model: 'gemini-2.0-flash',
      systemInstruction: 'You are a React component generator. When given a description, generate a single React component using JSX and Tailwind CSS classes. The code should be raw JSX (a single root element), with no imports, no exports, and no function wrapper. Use realistic placeholder content. Also generate a short descriptive title for the component.',
    });

    const result = await model.generateContent({
      contents: [
        {
          role: 'user',
          parts: [{ text: prompt }],
        },
      ],
      generationConfig: {
        temperature: 0.7,
        maxOutputTokens: 2000,
        responseMimeType: 'application/json',
        responseSchema: {
          type: SchemaType.OBJECT,
          properties: {
            title: {
              type: SchemaType.STRING,
              description: 'A short descriptive title for the component (3-6 words)',
            },
            code: {
              type: SchemaType.STRING,
              description: 'Raw JSX code for the component. Single root element. Use Tailwind CSS classes. No imports, exports, or function wrappers.',
            },
          },
          required: ['title', 'code'],
        },
      },
    });

    const raw = result.response.text();
    const parsed = JSON.parse(raw) as { title: string; code: string };

    if (!parsed.code) {
      setGenerationState({ status: 'error', message: 'No code was generated. Try a different prompt.' });
      return;
    }

    setGenerationState({
      status: 'success',
      code: parsed.code,
      prompt,
      title: parsed.title,
    });
  } catch (err) {
    const message = err instanceof Error ? err.message : 'Generation failed';
    setGenerationState({ status: 'error', message });
  }
}, [apiKey]);
```

### What Changed

1. **Added `responseMimeType: 'application/json'`** - Gemini now returns valid JSON
2. **Added `responseSchema`** - defines the shape: `{ title: string, code: string }`
3. **Parse with `JSON.parse`** instead of regex cleaning
4. **Moved system prompt** to `systemInstruction` in model config (cleaner)
5. **No more `cleanGeneratedCode` function** - delete it entirely
6. **No more `extractTitle` function** - the AI generates the title for us

---

## Step-by-Step Changes

### 1. Update `src/types.ts`

Add `title` to the success state so we can pass the AI-generated title around:

```typescript
// src/types.ts

// updated: success state now includes title from AI
export type GenerationState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; code: string; prompt: string; title: string }
  | { status: 'error'; message: string };
```

### 2. Update `src/App.tsx`

Remove the two helper functions and update the API call:

```typescript
// DELETE these two functions entirely:
// - cleanGeneratedCode
// - extractTitle

// UPDATE the import:
import { GoogleGenerativeAI, SchemaType } from '@google/generative-ai';

// UPDATE handleGenerate (shown above)

// UPDATE handleSave to use the AI-generated title:
const handleSave = useCallback(async () => {
  if (generationState.status !== 'success') return;
  if (!isFirebaseConfigured()) return;
  setIsSaving(true);
  try {
    await saveComponent(generationState.prompt, generationState.code, generationState.title);
    await fetchGallery();
  } catch (err) {
    console.error('failed to save component:', err);
  } finally {
    setIsSaving(false);
  }
}, [generationState, fetchGallery]);
```

Notice `handleSave` now uses `generationState.title` instead of `extractTitle(generationState.prompt)`.

### 3. No Changes Needed

These files don't need any changes:
- `src/preview-panel.tsx` - still receives `state.code` the same way
- `src/gallery.tsx` - still displays `component.title` the same way
- `src/firebase.ts` - still saves `title` field the same way
- `src/prompt-input.tsx` - still calls `onGenerate(prompt)` the same way

This is the beauty of good architecture - changing the AI integration doesn't affect the rest of the app.

---

## Understanding the Schema

```typescript
responseSchema: {
  type: SchemaType.OBJECT,
  properties: {
    title: {
      type: SchemaType.STRING,
      description: 'A short descriptive title for the component (3-6 words)',
    },
    code: {
      type: SchemaType.STRING,
      description: 'Raw JSX code for the component...',
    },
  },
  required: ['title', 'code'],
}
```

| Field | Purpose |
|-------|---------|
| `type: SchemaType.OBJECT` | The response is a JSON object |
| `properties` | Defines each field in the object |
| `type: SchemaType.STRING` | Each field is a string |
| `description` | Tells the AI what to put in each field |
| `required` | Both fields must be present |

The `description` fields are important - they act as instructions for the AI. Writing "Raw JSX code... No imports, exports, or function wrappers" in the `code` description ensures the AI puts clean JSX in that field.

### Available Schema Types

```typescript
SchemaType.STRING   // text
SchemaType.NUMBER   // numbers
SchemaType.BOOLEAN  // true/false
SchemaType.OBJECT   // nested object
SchemaType.ARRAY    // array of items
```

You could extend the schema to include more fields:

```typescript
// example: generating components with metadata
responseSchema: {
  type: SchemaType.OBJECT,
  properties: {
    title: { type: SchemaType.STRING },
    code: { type: SchemaType.STRING },
    description: { type: SchemaType.STRING },
    tags: {
      type: SchemaType.ARRAY,
      items: { type: SchemaType.STRING },
    },
  },
  required: ['title', 'code'],
}
```

---

## Before and After Comparison

### Before (Classes 3-8): Free-form text + regex cleaning

```typescript
// 15 lines of regex cleaning code
const cleanGeneratedCode = (raw: string): string => {
  let code = raw.trim();
  code = code.replace(/^```(?:jsx|tsx|javascript|typescript)?\s*\n?/i, '');
  code = code.replace(/\n?```\s*$/i, '');
  code = code.replace(/^import\s+.*;\s*\n?/gm, '');
  code = code.replace(/^export\s+(default\s+)?/gm, '');
  const fnMatch = code.match(/...complex regex.../);
  if (fnMatch?.[1]) { code = fnMatch[1].trim(); }
  return code.trim();
};

// 5 lines for title extraction
const extractTitle = (prompt: string): string => {
  const words = prompt.split(/\s+/).slice(0, 6).join(' ');
  return words.length > 50 ? words.slice(0, 50) + '...' : words;
};

// API call returns raw text
const raw = result.response.text();
const code = cleanGeneratedCode(raw);
const title = extractTitle(prompt); // just first 6 words of prompt
```

### After (Class 9): Structured output

```typescript
// no cleaning functions needed

// API call returns structured JSON
const raw = result.response.text();
const parsed = JSON.parse(raw) as { title: string; code: string };
const code = parsed.code;   // already clean
const title = parsed.title; // AI-generated, descriptive
```

**Lines of code removed**: ~20
**Reliability improvement**: Significant - no more regex edge cases
**Title quality**: AI generates "Dark Mode Pricing Card with Toggle" instead of "A dark pricing card with"

---

## Error Handling for JSON Parsing

Since we're now parsing JSON, we should handle the case where the response isn't valid JSON:

```typescript
try {
  const raw = result.response.text();
  const parsed = JSON.parse(raw) as { title: string; code: string };
  // ... use parsed.title and parsed.code
} catch (err) {
  if (err instanceof SyntaxError) {
    // JSON parsing failed - the AI returned invalid JSON
    setGenerationState({ status: 'error', message: 'AI returned invalid response. Try again.' });
    return;
  }
  // other errors (network, API, etc.)
  const message = err instanceof Error ? err.message : 'Generation failed';
  setGenerationState({ status: 'error', message });
}
```

In practice, when you use `responseMimeType: 'application/json'` with a schema, Gemini almost always returns valid JSON. But it's good practice to handle the edge case.

---

## Real-World Use Cases for Structured Outputs

Structured outputs aren't just for our component builder. They're used everywhere:

1. **Chatbots**: Extract intent and entities from user messages
   ```json
   { "intent": "book_flight", "from": "NYC", "to": "London", "date": "2026-04-15" }
   ```

2. **Content generation**: Get articles with metadata
   ```json
   { "title": "...", "summary": "...", "body": "...", "tags": ["react", "tutorial"] }
   ```

3. **Data extraction**: Parse unstructured text into structured data
   ```json
   { "name": "John Smith", "email": "john@example.com", "company": "Acme Inc" }
   ```

4. **Code review**: Get structured feedback
   ```json
   { "issues": [{ "line": 15, "severity": "warning", "message": "..." }] }
   ```

---

## Key Concepts Recap

**Structured Outputs** tell AI models to return data in a specific JSON format. You define a schema, and the model guarantees its output matches that schema.

**Schema Design** is about choosing the right fields and writing good descriptions. The `description` field acts as an instruction to the AI about what to put in each field.

**JSON.parse** converts a JSON string into a JavaScript object. Always wrap it in try/catch because malformed strings will throw a `SyntaxError`.

**Type Assertion** (`as { title: string; code: string }`) tells TypeScript the shape of the parsed JSON. Since `JSON.parse` returns `any`, the assertion gives you type safety.

**Separation of Concerns** - Because we designed our app with clean interfaces (props, types, state), changing how the AI returns data only affected App.tsx. The preview, gallery, and sidebar components didn't need any changes.

---

## What's Next

In Class 10, you'll choose your own React project to build over the next 15 days, and set up your portfolio website with a custom domain to showcase your work.
