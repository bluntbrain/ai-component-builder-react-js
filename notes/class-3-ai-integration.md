# Class 3: AI Integration - OpenAI API

## What You'll Learn

- How the OpenAI Chat Completions API works
- The difference between system messages and user messages
- Prompt engineering - how to write instructions that make AI return usable code
- Async/await patterns for API calls in React
- Error handling strategies for external APIs
- Cleaning AI-generated code with regular expressions
- State transitions in React (idle -> loading -> success/error)

## What You'll Build

By the end of this class, you'll be able to type a component description (like "a dark pricing card with a toggle"), click Generate, and receive raw JSX code from OpenAI. The code will be cleaned and stored in React state, ready for the live preview we'll build in Class 4.

---

## Step 1: Install and Understand the OpenAI SDK

We already installed the `openai` package in Class 1. Here's how it works:

The `openai` npm package is the official SDK from OpenAI. It provides a typed JavaScript/TypeScript client for all OpenAI APIs. We'll use the **Chat Completions** endpoint, which is the same API behind ChatGPT.

### How Chat Completions Work

You send an array of messages, each with a `role`:

| Role | Purpose |
|------|---------|
| `system` | Instructions for the AI. Sets behavior, tone, and constraints. The AI treats this as its "programming" |
| `user` | The human's message. What the user is asking for |
| `assistant` | The AI's response. Used when providing conversation history |

The AI responds with a new `assistant` message based on the full conversation context.

## Step 1.5: Try the API in Postman First

Before writing any code, let's call the OpenAI API directly from Postman. This helps you understand exactly what the API expects and what it returns - without React, TypeScript, or any SDK getting in the way.

### Why Postman First?

When something breaks in your app, you need to know: is it the API or my code? If you've already seen the API work in Postman, you know the API side is fine and the bug is in your React code. This is a debugging superpower.

### Setting Up the Request

1. Open Postman (download from [postman.com](https://www.postman.com/downloads/) if you don't have it)
2. Create a new **POST** request
3. Set the URL to:

```
https://api.openai.com/v1/chat/completions
```

### Headers

Add these two headers:

| Key | Value |
|-----|-------|
| `Content-Type` | `application/json` |
| `Authorization` | `Bearer sk-your-api-key-here` |

Note the `Bearer ` prefix (with a space) before your API key. This is standard for API authentication.

### Request Body

Click the **Body** tab, select **raw** and **JSON**, then paste:

```json
{
  "model": "gpt-4o",
  "messages": [
    {
      "role": "system",
      "content": "Return only raw JSX for a single React component. No imports, no exports, no function wrapper, no explanations, no markdown code fences. Use only Tailwind CSS classes for styling. The JSX should be a single root element. Use realistic placeholder content."
    },
    {
      "role": "user",
      "content": "A dark pricing card with monthly and annual toggle"
    }
  ],
  "temperature": 0.7,
  "max_tokens": 2000
}
```

### Send It

Click **Send**. You'll get a response like this:

```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1711234567,
  "model": "gpt-4o-2024-08-06",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "<div className=\"bg-gray-900 rounded-2xl p-8 max-w-sm mx-auto\">\n  <h2 className=\"text-2xl font-bold text-white\">Pro Plan</h2>\n  ..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 85,
    "completion_tokens": 342,
    "total_tokens": 427
  }
}
```

### What to Notice

**The response structure:**
- `choices` is an array (usually with 1 item)
- `choices[0].message.content` contains the AI's text response - this is the JSX we need
- `usage` tells you how many tokens were used (affects billing)

**The content field** is where our generated JSX lives. In code, we'll access it as:
```typescript
response.choices[0].message.content
```

**Experiment with it:**
- Change the `user` message to different component descriptions
- Try changing `temperature` to `0` (very predictable) vs `1` (very creative)
- Try lowering `max_tokens` to `500` and see how it cuts off longer responses
- Remove the `system` message and see how the output changes (it'll include markdown fences, imports, etc.)

### Understanding the API Flow

```
Your App                         OpenAI API
   |                                |
   |--- POST /chat/completions ---->|
   |    (model, messages, config)   |
   |                                |
   |<--- JSON response -------------|
   |    (choices[0].message.content) |
   |                                |
   v                                v
Parse the content, clean it, render it
```

This is exactly what our React code will do - just with the `openai` npm package handling the HTTP request for us instead of Postman.

---

## Step 2: Write the Code Cleaning Function

AI models often return code wrapped in markdown fences, with import statements, or inside function declarations. We need to strip all that out to get pure JSX.

Add this to `src/App.tsx` (above the `App` component):

```typescript
// strip markdown fences and import/export lines from ai response
const cleanGeneratedCode = (raw: string): string => {
  let code = raw.trim();
  // remove markdown code fences like ```jsx ... ```
  code = code.replace(/^```(?:jsx|tsx|javascript|typescript)?\s*\n?/i, '');
  code = code.replace(/\n?```\s*$/i, '');
  // remove import/export lines
  code = code.replace(/^import\s+.*;\s*\n?/gm, '');
  code = code.replace(/^export\s+(default\s+)?/gm, '');
  // remove function/const component wrappers if AI wraps in a component definition
  const fnMatch = code.match(/(?:function|const)\s+\w+\s*(?:=\s*)?(?:\([^)]*\)\s*(?:=>)?\s*)?[({]\s*\n?\s*return\s*\(\s*\n?([\s\S]*?)\n?\s*\)\s*;?\s*\n?\s*[})]\s*;?\s*$/);
  if (fnMatch?.[1]) {
    code = fnMatch[1].trim();
  }
  return code.trim();
};
```

### How the Cleaning Works

The AI might return any of these formats:

**Format 1: Markdown fenced**
````
```jsx
<div className="p-4">Hello</div>
```
````

**Format 2: With imports**
```
import React from 'react';
export default function Card() {
  return (
    <div className="p-4">Hello</div>
  );
}
```

**Format 3: Clean JSX (what we want)**
```
<div className="p-4">Hello</div>
```

The regex pipeline transforms any format into Format 3:

1. `code.replace(/^```...$/i, '')` - strips the markdown fence markers
2. `code.replace(/^import\s+.*;\s*\n?/gm, '')` - removes all import lines
3. `code.replace(/^export\s+(default\s+)?/gm, '')` - removes export keywords
4. The `fnMatch` regex detects if the code is wrapped in a function like `function Card() { return (...) }` and extracts just the JSX inside the return statement

### Key Concept: Regular Expressions

If you're new to regex, here's a breakdown of the markdown fence pattern:

```
/^```(?:jsx|tsx|javascript|typescript)?\s*\n?/i
  ^     start of string
  ```   literal backticks
  (?:)  non-capturing group (don't save match)
  ?     the language tag is optional
  \s*   any whitespace
  \n?   optional newline
  /i    case-insensitive flag
```

The `g` and `m` flags on the import regex:
- `g` = global (replace ALL matches, not just the first)
- `m` = multiline (^ matches start of each line, not just start of string)

## Step 3: Write the Title Extractor

We'll need a short title for each saved component. Instead of making another API call, we'll extract it from the prompt:

```typescript
// extract a short title from the prompt
const extractTitle = (prompt: string): string => {
  const words = prompt.split(/\s+/).slice(0, 6).join(' ');
  return words.length > 50 ? words.slice(0, 50) + '...' : words;
};
```

This takes the first 6 words and caps at 50 characters. Simple and fast.

## Step 4: Update App.tsx with OpenAI Integration

Replace your entire `src/App.tsx`:

```typescript
// src/App.tsx
// purpose: main app shell with state management, openai integration, and 3-panel layout

import { useState, useEffect, useCallback } from 'react';
import OpenAI from 'openai';
import type { GenerationState, GalleryState } from './types';
import { Sidebar } from './prompt-input';

// strip markdown fences and import/export lines from ai response
const cleanGeneratedCode = (raw: string): string => {
  let code = raw.trim();
  code = code.replace(/^```(?:jsx|tsx|javascript|typescript)?\s*\n?/i, '');
  code = code.replace(/\n?```\s*$/i, '');
  code = code.replace(/^import\s+.*;\s*\n?/gm, '');
  code = code.replace(/^export\s+(default\s+)?/gm, '');
  const fnMatch = code.match(/(?:function|const)\s+\w+\s*(?:=\s*)?(?:\([^)]*\)\s*(?:=>)?\s*)?[({]\s*\n?\s*return\s*\(\s*\n?([\s\S]*?)\n?\s*\)\s*;?\s*\n?\s*[})]\s*;?\s*$/);
  if (fnMatch?.[1]) {
    code = fnMatch[1].trim();
  }
  return code.trim();
};

const extractTitle = (prompt: string): string => {
  const words = prompt.split(/\s+/).slice(0, 6).join(' ');
  return words.length > 50 ? words.slice(0, 50) + '...' : words;
};

export const App = () => {
  const [apiKey, setApiKey] = useState(() => localStorage.getItem('openai_api_key') ?? '');
  const [generationState, setGenerationState] = useState<GenerationState>({ status: 'idle' });
  const [galleryState] = useState<GalleryState>({ status: 'idle' });
  const [isSaving] = useState(false);

  const handleGenerate = useCallback(async (prompt: string) => {
    if (!apiKey) return;
    setGenerationState({ status: 'loading' });

    try {
      const openai = new OpenAI({
        apiKey,
        dangerouslyAllowBrowser: true,
      });

      const response = await openai.chat.completions.create({
        model: 'gpt-4o',
        messages: [
          {
            role: 'system',
            content:
              'Return only raw JSX for a single React component. No imports, no exports, no function wrapper, no explanations, no markdown code fences. Use only Tailwind CSS classes for styling. The JSX should be a single root element. Use realistic placeholder content.',
          },
          { role: 'user', content: prompt },
        ],
        temperature: 0.7,
        max_tokens: 2000,
      });

      const raw = response.choices[0]?.message?.content ?? '';
      const code = cleanGeneratedCode(raw);

      if (!code) {
        setGenerationState({ status: 'error', message: 'No code was generated. Try a different prompt.' });
        return;
      }

      setGenerationState({ status: 'success', code, prompt });
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Generation failed';
      setGenerationState({ status: 'error', message });
    }
  }, [apiKey]);

  return (
    <div className="flex h-screen bg-gray-950 text-white overflow-hidden">
      {/* left sidebar */}
      <Sidebar
        onGenerate={handleGenerate}
        isLoading={generationState.status === 'loading'}
        apiKey={apiKey}
        onApiKeySave={setApiKey}
      />

      {/* center - shows state for now */}
      <div className="flex-1 flex items-center justify-center p-8">
        {generationState.status === 'idle' && (
          <p className="text-gray-500">Describe a component to generate code</p>
        )}
        {generationState.status === 'loading' && (
          <div className="flex flex-col items-center gap-3">
            <div className="w-8 h-8 border-2 border-violet-500 border-t-transparent rounded-full animate-spin" />
            <p className="text-sm text-gray-400">Generating...</p>
          </div>
        )}
        {generationState.status === 'error' && (
          <p className="text-red-400">{generationState.message}</p>
        )}
        {generationState.status === 'success' && (
          <div className="max-w-2xl w-full">
            <p className="text-sm text-gray-400 mb-2">Generated code (preview coming in Class 4):</p>
            <pre className="bg-gray-900 p-4 rounded-lg text-sm text-green-400 overflow-auto max-h-96">
              {generationState.code}
            </pre>
          </div>
        )}
      </div>

      {/* right sidebar placeholder */}
      <aside className="w-48 bg-gray-900 border-l border-gray-800 flex items-center justify-center">
        <p className="text-xs text-gray-500">Gallery - Class 5</p>
      </aside>
    </div>
  );
};
```

### How the API Call Works

Let's break down `handleGenerate` step by step:

**1. Guard clause:**
```typescript
if (!apiKey) return;
```
Don't call the API if no key is set.

**2. Set loading state:**
```typescript
setGenerationState({ status: 'loading' });
```
This immediately updates the UI to show a spinner.

**3. Create OpenAI client:**
```typescript
const openai = new OpenAI({
  apiKey,
  dangerouslyAllowBrowser: true,
});
```
`dangerouslyAllowBrowser: true` is required because the OpenAI SDK is designed for server-side use. This flag acknowledges that we're intentionally using it in the browser. The API key is provided by the user themselves, so this is acceptable for a demo project.

**4. Make the API call:**
```typescript
const response = await openai.chat.completions.create({
  model: 'gpt-4o',
  messages: [...],
  temperature: 0.7,
  max_tokens: 2000,
});
```

| Parameter | Value | Why |
|-----------|-------|-----|
| `model` | `'gpt-4o'` | GPT-4o is fast and good at code generation |
| `temperature` | `0.7` | Controls randomness. 0 = deterministic, 1 = creative. 0.7 balances variety with correctness |
| `max_tokens` | `2000` | Limits response length. A typical component is 500-1500 tokens |

**5. Extract the response:**
```typescript
const raw = response.choices[0]?.message?.content ?? '';
```
The API returns an array of `choices` (usually just one). We use optional chaining (`?.`) because any part could theoretically be null. The `??` provides a fallback empty string.

**6. Clean and validate:**
```typescript
const code = cleanGeneratedCode(raw);
if (!code) {
  setGenerationState({ status: 'error', message: 'No code was generated.' });
  return;
}
```

**7. Set success state:**
```typescript
setGenerationState({ status: 'success', code, prompt });
```
We store both the cleaned code AND the original prompt (we'll need it when saving to the database).

**8. Error handling:**
```typescript
catch (err) {
  const message = err instanceof Error ? err.message : 'Generation failed';
  setGenerationState({ status: 'error', message });
}
```
`err instanceof Error` is a **type guard**. It checks if the caught value is an Error object (which has a `.message` property). If not, we use a generic fallback string. This is necessary because TypeScript types `catch` errors as `unknown`.

### Key Concept: Prompt Engineering

The system message is carefully crafted:

```
Return only raw JSX for a single React component.
No imports, no exports, no function wrapper, no explanations, no markdown code fences.
Use only Tailwind CSS classes for styling.
The JSX should be a single root element.
Use realistic placeholder content.
```

Each instruction solves a specific problem:
- "raw JSX" - we don't want a complete file, just the markup
- "No imports/exports/function wrapper" - reduces cleaning work
- "no markdown code fences" - prevents ``` wrapping
- "Tailwind CSS classes" - matches our styling system
- "single root element" - JSX requires a single parent element
- "realistic placeholder content" - makes the preview look real (actual names, prices, etc.)

## Step 5: Test the Integration

1. Run `npm run dev`
2. Enter your OpenAI API key in the sidebar and click Save
3. Type "A dark pricing card with monthly/annual toggle" in the textarea
4. Click "Generate Component"
5. Watch the state transitions: idle -> loading (spinner) -> success (code displayed)
6. Try an example chip - it auto-fills and generates

If you see an error, check:
- Is your API key correct?
- Does your OpenAI account have credits?
- Check the browser console for detailed error messages

---

## Key Concepts Recap

**async/await** - Modern syntax for handling asynchronous operations (like API calls). `await` pauses execution until the Promise resolves. The function containing `await` must be marked `async`.

**try/catch** - Error handling for async operations. If the `await` call fails (network error, API error, etc.), execution jumps to the `catch` block.

**State Transitions** - Using discriminated unions to represent the current state of an async operation. The UI renders different content based on `state.status`. This makes it impossible to show a spinner and an error at the same time.

**Type Guards** - `err instanceof Error` narrows the type from `unknown` to `Error`, giving you access to `.message`. TypeScript enforces this - you can't access `.message` on `unknown`.

**Prompt Engineering** - Writing instructions for AI models. Be specific about format, constraints, and what NOT to include. Test your prompts and iterate.

---

## What's Next

In Class 4, we'll take the generated code and render it live in a sandboxed iframe. We'll also add syntax highlighting with line numbers and a copy-to-clipboard button.
