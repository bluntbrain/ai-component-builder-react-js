# Class 8: Using Gemini API (Free Alternative to OpenAI)

## What You'll Learn

- Why Google Gemini is a great free alternative for students
- How to get a free Gemini API key
- Swapping the OpenAI SDK for Google's Generative AI SDK
- The differences between OpenAI and Gemini API calls
- Updating the UI to reflect the change
- A side-by-side comparison of both approaches

## Why Switch to Gemini?

OpenAI charges for API usage. Even with a free trial, credits run out fast when you're learning and experimenting. Google's Gemini API offers a **free tier** that's generous enough for development and learning:

- Free API key from Google AI Studio
- Good code generation quality (Gemini 2.0 Flash is fast and capable)
- No credit card required
- Enough free requests for building and testing projects

---

## Step 1: Get a Free Gemini API Key

1. Go to [Google AI Studio](https://aistudio.google.com)
2. Sign in with your Google account
3. Click "Get API Key" in the left sidebar
4. Click "Create API Key"
5. Copy the key (starts with `AIza...`)

Save this key somewhere safe. You'll add it to your app in a moment.

---

## Step 2: Install the Gemini SDK

Remove the OpenAI package and install Google's Generative AI SDK:

```bash
npm uninstall openai
npm install @google/generative-ai
```

Your `package.json` dependencies will now show `@google/generative-ai` instead of `openai`.

---

## Step 3: Update App.tsx

This is the main change. We replace the OpenAI client with Gemini.

### Before (OpenAI)

```typescript
import OpenAI from 'openai';

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
          content: 'Return only raw JSX for a single React component...',
        },
        { role: 'user', content: prompt },
      ],
      temperature: 0.7,
      max_tokens: 2000,
    });

    const raw = response.choices[0]?.message?.content ?? '';
    // ...
  } catch (err) { /* ... */ }
}, [apiKey]);
```

### After (Gemini)

```typescript
import { GoogleGenerativeAI } from '@google/generative-ai';

const handleGenerate = useCallback(async (prompt: string) => {
  if (!apiKey) return;
  setGenerationState({ status: 'loading' });

  try {
    const genAI = new GoogleGenerativeAI(apiKey);
    const model = genAI.getGenerativeModel({ model: 'gemini-2.0-flash' });

    const systemPrompt = 'Return only raw JSX for a single React component. No imports, no exports, no function wrapper, no explanations, no markdown code fences. Use only Tailwind CSS classes for styling. The JSX should be a single root element. Use realistic placeholder content.';

    const result = await model.generateContent({
      contents: [
        {
          role: 'user',
          parts: [{ text: `${systemPrompt}\n\nUser request: ${prompt}` }],
        },
      ],
      generationConfig: {
        temperature: 0.7,
        maxOutputTokens: 2000,
      },
    });

    const raw = result.response.text();
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
```

### Key Differences

| Aspect | OpenAI | Gemini |
|--------|--------|--------|
| Package | `openai` | `@google/generative-ai` |
| Client | `new OpenAI({ apiKey })` | `new GoogleGenerativeAI(apiKey)` |
| Model selection | In the API call (`model: 'gpt-4o'`) | `genAI.getGenerativeModel({ model: '...' })` |
| System prompt | Separate `system` role message | Combined with user message (Gemini handles system instructions differently) |
| API call | `chat.completions.create()` | `model.generateContent()` |
| Response | `response.choices[0]?.message?.content` | `result.response.text()` |
| Browser flag | Needs `dangerouslyAllowBrowser: true` | Works in browser by default |
| Temperature | `temperature: 0.7` | `temperature: 0.7` (same) |
| Max length | `max_tokens: 2000` | `maxOutputTokens: 2000` |

### What Stays the Same

- The `cleanGeneratedCode` function - works the same regardless of AI provider
- The `GenerationState` discriminated union - same state management
- The `extractTitle` function - same logic
- Error handling pattern - same try/catch structure
- All other components (preview panel, gallery, sidebar UI) - unchanged

---

## Step 4: Update the Full App.tsx

Here's the complete updated `src/App.tsx`:

```typescript
// src/App.tsx
// purpose: main app shell with state management, gemini integration, and 3-panel layout

import { useState, useEffect, useCallback } from 'react';
import { GoogleGenerativeAI } from '@google/generative-ai';
import type { GenerationState, GalleryState } from './types';
import { isFirebaseConfigured, saveComponent, listComponents } from './firebase';
import { Sidebar } from './prompt-input';
import { PreviewPanel } from './preview-panel';
import { VariantsSidebar } from './gallery';

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

const SYSTEM_PROMPT = 'Return only raw JSX for a single React component. No imports, no exports, no function wrapper, no explanations, no markdown code fences. Use only Tailwind CSS classes for styling. The JSX should be a single root element. Use realistic placeholder content.';

export const App = () => {
  const [apiKey, setApiKey] = useState(() => localStorage.getItem('gemini_api_key') ?? '');
  const [generationState, setGenerationState] = useState<GenerationState>({ status: 'idle' });
  const [galleryState, setGalleryState] = useState<GalleryState>({ status: 'idle' });
  const [isSaving, setIsSaving] = useState(false);

  const fetchGallery = useCallback(async () => {
    if (!isFirebaseConfigured()) return;
    setGalleryState({ status: 'loading' });
    try {
      const components = await listComponents();
      setGalleryState({ status: 'success', components });
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Failed to load gallery';
      setGalleryState({ status: 'error', message });
    }
  }, []);

  useEffect(() => {
    fetchGallery();
  }, [fetchGallery]);

  const handleGenerate = useCallback(async (prompt: string) => {
    if (!apiKey) return;
    setGenerationState({ status: 'loading' });

    try {
      const genAI = new GoogleGenerativeAI(apiKey);
      const model = genAI.getGenerativeModel({ model: 'gemini-2.0-flash' });

      const result = await model.generateContent({
        contents: [
          {
            role: 'user',
            parts: [{ text: `${SYSTEM_PROMPT}\n\nUser request: ${prompt}` }],
          },
        ],
        generationConfig: {
          temperature: 0.7,
          maxOutputTokens: 2000,
        },
      });

      const raw = result.response.text();
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

  const handleSave = useCallback(async () => {
    if (generationState.status !== 'success') return;
    if (!isFirebaseConfigured()) return;
    setIsSaving(true);
    try {
      const title = extractTitle(generationState.prompt);
      await saveComponent(generationState.prompt, generationState.code, title);
      await fetchGallery();
    } catch (err) {
      console.error('failed to save component:', err);
    } finally {
      setIsSaving(false);
    }
  }, [generationState, fetchGallery]);

  return (
    <div className="flex h-screen bg-gray-950 text-white overflow-hidden">
      <Sidebar
        onGenerate={handleGenerate}
        isLoading={generationState.status === 'loading'}
        apiKey={apiKey}
        onApiKeySave={setApiKey}
      />
      <PreviewPanel
        state={generationState}
        onSave={handleSave}
        isSaving={isSaving}
      />
      <VariantsSidebar state={galleryState} onRefresh={fetchGallery} />
    </div>
  );
};
```

---

## Step 5: Update the Sidebar Labels

In `src/prompt-input.tsx`, update the API key section to say "Gemini" instead of "OpenAI":

```typescript
// change the label
<label className="block text-xs font-medium text-gray-400 mb-1.5">Gemini API Key</label>

// change the localStorage key
onClick={() => {
  localStorage.setItem('gemini_api_key', keyValue);
  onApiKeySave(keyValue);
}}

// change the footer text
<p className="text-xs text-gray-600">Free API key from Google AI Studio</p>
```

Also update the `useState` initializer in App.tsx:

```typescript
const [apiKey, setApiKey] = useState(() => localStorage.getItem('gemini_api_key') ?? '');
```

---

## Step 6: Test

1. Run `npm run dev`
2. Enter your Gemini API key in the sidebar
3. Click Save
4. Type a prompt like "A dark pricing card with monthly and annual toggle"
5. Click Generate
6. The component should render in the preview just like before

If you get an error, check:
- Is your API key correct?
- Go to Google AI Studio and verify the key is active
- Check the browser console for the full error message

---

## Understanding the Gemini API

### The `contents` Array

Gemini uses a `contents` array instead of a `messages` array:

```typescript
contents: [
  {
    role: 'user',
    parts: [{ text: 'your message here' }],
  },
]
```

Each message has a `role` (`user` or `model`) and `parts` (array of content parts - text, images, etc.).

### System Instructions

Gemini handles system prompts differently from OpenAI. Instead of a separate `system` role, you can either:

1. Prepend the system instructions to the user message (what we do above)
2. Use the `systemInstruction` field in model config:

```typescript
const model = genAI.getGenerativeModel({
  model: 'gemini-2.0-flash',
  systemInstruction: 'Return only raw JSX...',
});
```

Both approaches work. We use option 1 for simplicity.

### Available Models

| Model | Speed | Quality | Best For |
|-------|-------|---------|----------|
| `gemini-2.0-flash` | Very fast | Good | Most tasks, free tier |
| `gemini-2.5-pro` | Slower | Best | Complex code generation |
| `gemini-2.5-flash` | Fast | Great | Balance of speed and quality |

For our use case, `gemini-2.0-flash` is the best choice - it's fast, free, and generates good JSX.

---

## Key Concepts Recap

**API Abstraction** - Because we separated our state management (discriminated unions) from the API call, switching providers only required changing the `handleGenerate` function. The rest of the app (preview, gallery, sidebar) didn't need any changes. This is good software design.

**System Prompts** - The same prompt engineering works across different AI models. "Return only raw JSX, no imports, no markdown" works with both OpenAI and Gemini.

**SDK Differences** - Different AI providers have different SDK shapes, but the concept is the same: send a prompt, get text back. The skill is reading documentation and adapting.

**Free Tier Mindset** - For learning and prototyping, always look for free alternatives. Gemini, Cloudflare Workers AI, and others offer free tiers that are more than enough for development.

---

## What's Next

In Class 9, we'll learn about Structured Outputs - a technique where we tell the AI to return JSON with a specific schema instead of raw text. This eliminates the need for the `cleanGeneratedCode` regex function entirely and makes the output more reliable.
