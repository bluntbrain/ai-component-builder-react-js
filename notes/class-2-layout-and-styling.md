# Class 2: Layout, Styling & Component Architecture

## What You'll Learn

- How to design TypeScript interfaces and discriminated unions for type-safe state
- Building a 3-panel layout with CSS Flexbox
- Creating a dark-themed UI with Tailwind CSS
- React component composition and props
- Controlled form inputs with `useState`
- Event handling with `useCallback`
- Conditional rendering patterns

## What You'll Build

A 3-panel application layout: a left sidebar with a prompt input form and example chips, a center panel (empty placeholder for now), and a right sidebar for the gallery. The UI will have a professional dark theme with interactive elements.

---

## Step 1: Define TypeScript Types

Before writing any components, we define all our data structures and state types. This is a "types-first" approach - it forces you to think about your data before your UI.

Create `src/types.ts`:

```typescript
// src/types.ts
// purpose: shared typescript interfaces and discriminated union state types

// firestore document model for saved components
export interface ComponentDocument {
  id: string;
  prompt: string;
  code: string;
  title: string;
  createdAt: number;
}

// discriminated union for ai generation state
export type GenerationState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; code: string; prompt: string }
  | { status: 'error'; message: string };

// discriminated union for gallery fetch state
export type GalleryState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; components: ComponentDocument[] }
  | { status: 'error'; message: string };

// component props
export interface PromptInputProps {
  onGenerate: (prompt: string) => void;
  isLoading: boolean;
}

export interface PreviewPanelProps {
  state: GenerationState;
  onSave: () => void;
  isSaving: boolean;
}

export interface GalleryGridProps {
  state: GalleryState;
  onRefresh: () => void;
}
```

### Key Concept: Discriminated Unions

A discriminated union is a TypeScript pattern where a type can be one of several shapes, and a shared field (the "discriminant") tells you which shape you're working with.

```typescript
type GenerationState =
  | { status: 'idle' }                                    // no extra fields
  | { status: 'loading' }                                 // no extra fields
  | { status: 'success'; code: string; prompt: string }   // has code + prompt
  | { status: 'error'; message: string };                  // has error message
```

Why is this powerful? When you check `state.status` in your code, TypeScript automatically narrows the type:

```typescript
if (state.status === 'success') {
  // TypeScript KNOWS state.code exists here
  console.log(state.code);   // no error
}

if (state.status === 'error') {
  // TypeScript KNOWS state.message exists here
  console.log(state.message); // no error
}

if (state.status === 'idle') {
  // TypeScript KNOWS there is NO code or message property
  console.log(state.code);    // ERROR! Property 'code' does not exist
}
```

This is much safer than using a single object with optional fields like `{ status: string; code?: string; error?: string }` because you can't accidentally access `code` when the status is `'error'`.

### Key Concept: Interface vs Type

- Use `interface` for object shapes (like `ComponentDocument`) - they can be extended
- Use `type` for unions and intersections (like `GenerationState`) - they can combine types

## Step 2: Build the Sidebar Component

Create `src/prompt-input.tsx`:

```typescript
// src/prompt-input.tsx
// purpose: left sidebar with prompt input, example chips, and api key config

import { useState, useCallback, type FormEvent } from 'react';
import type { PromptInputProps } from './types';

const EXAMPLE_PROMPTS = [
  'A dark pricing card with monthly/annual toggle',
  'A user profile card with avatar and social links',
  'A notification toast with progress bar',
  'A login form with email and password',
  'A testimonial card with star ratings',
  'A stats dashboard card with charts',
];

interface SidebarProps extends PromptInputProps {
  apiKey: string;
  onApiKeySave: (key: string) => void;
}

export const Sidebar = ({
  onGenerate,
  isLoading,
  apiKey,
  onApiKeySave,
}: SidebarProps) => {
  const [input, setInput] = useState('');
  const [keyValue, setKeyValue] = useState(apiKey);
  const [showKey, setShowKey] = useState(false);

  const handleSubmit = useCallback(
    (e: FormEvent) => {
      e.preventDefault();
      if (!input.trim() || isLoading) return;
      onGenerate(input.trim());
    },
    [input, isLoading, onGenerate],
  );

  const handleChipClick = useCallback(
    (prompt: string) => {
      setInput(prompt);
      if (!isLoading) onGenerate(prompt);
    },
    [isLoading, onGenerate],
  );

  return (
    <aside className="w-72 bg-gray-900 border-r border-gray-800 flex flex-col h-screen sticky top-0">
      {/* logo */}
      <div className="px-4 py-4 border-b border-gray-800">
        <div className="flex items-center gap-2.5">
          <div className="w-7 h-7 bg-linear-to-br from-violet-500 to-indigo-600 rounded-lg" />
          <span className="font-semibold text-white text-sm">AI Component Builder</span>
        </div>
      </div>

      {/* api key */}
      <div className="px-4 py-3 border-b border-gray-800">
        <label className="block text-xs font-medium text-gray-400 mb-1.5">OpenAI API Key</label>
        <div className="flex gap-1.5">
          <input
            type={showKey ? 'text' : 'password'}
            value={keyValue}
            onChange={(e) => setKeyValue(e.target.value)}
            placeholder="sk-..."
            className="flex-1 min-w-0 bg-gray-800 border border-gray-700 rounded-lg px-2.5 py-1.5 text-xs text-white placeholder-gray-500 focus:outline-none focus:ring-1 focus:ring-violet-500"
          />
          <button
            onClick={() => setShowKey(!showKey)}
            className="px-2 py-1.5 text-xs text-gray-400 bg-gray-800 border border-gray-700 rounded-lg hover:text-white transition-colors"
          >
            {showKey ? 'Hide' : 'Show'}
          </button>
          <button
            onClick={() => {
              localStorage.setItem('openai_api_key', keyValue);
              onApiKeySave(keyValue);
            }}
            className="px-2.5 py-1.5 text-xs font-medium text-white bg-violet-600 rounded-lg hover:bg-violet-700 transition-colors"
          >
            Save
          </button>
        </div>
      </div>

      {/* prompt input */}
      <div className="flex-1 overflow-y-auto px-4 py-4 space-y-4">
        <form onSubmit={handleSubmit} className="space-y-3">
          <textarea
            value={input}
            onChange={(e) => setInput(e.target.value)}
            placeholder="Describe a UI component..."
            rows={5}
            className="w-full bg-gray-800 border border-gray-700 rounded-lg px-3 py-2.5 text-sm text-white placeholder-gray-500 resize-none focus:outline-none focus:ring-1 focus:ring-violet-500"
          />
          <button
            type="submit"
            disabled={!input.trim() || isLoading}
            className="w-full py-2.5 text-sm font-medium text-white bg-linear-to-r from-violet-600 to-indigo-600 rounded-lg hover:from-violet-700 hover:to-indigo-700 disabled:opacity-50 disabled:cursor-not-allowed transition-all"
          >
            {isLoading ? (
              <span className="flex items-center justify-center gap-2">
                <span className="w-3.5 h-3.5 border-2 border-white border-t-transparent rounded-full animate-spin" />
                Generating...
              </span>
            ) : (
              'Generate Component'
            )}
          </button>
        </form>

        {/* example chips */}
        <div>
          <p className="text-xs text-gray-500 mb-2">Try an example:</p>
          <div className="flex flex-col gap-1.5">
            {EXAMPLE_PROMPTS.map((prompt) => (
              <button
                key={prompt}
                onClick={() => handleChipClick(prompt)}
                disabled={isLoading}
                className="text-left text-xs px-3 py-2 bg-gray-800/50 text-gray-400 rounded-lg border border-gray-800 hover:border-gray-700 hover:text-gray-200 disabled:opacity-50 transition-colors"
              >
                {prompt}
              </button>
            ))}
          </div>
        </div>
      </div>

      {/* footer */}
      <div className="px-4 py-3 border-t border-gray-800">
        <p className="text-xs text-gray-600">Key stored locally in browser only</p>
      </div>
    </aside>
  );
};
```

### How It Works

**Controlled Inputs**: The textarea and API key input are "controlled components". Their values come from React state (`useState`), and every keystroke triggers `onChange` which updates the state. React re-renders the component, and the input shows the new value. This gives you full control over form data.

```typescript
const [input, setInput] = useState('');
// ...
<textarea
  value={input}                           // value comes from state
  onChange={(e) => setInput(e.target.value)} // state updates on every keystroke
/>
```

**useCallback**: Wraps functions that get passed as props or used in dependency arrays. Without `useCallback`, a new function reference is created on every render, which can cause unnecessary re-renders in child components.

```typescript
const handleSubmit = useCallback(
  (e: FormEvent) => {
    e.preventDefault();
    if (!input.trim() || isLoading) return;
    onGenerate(input.trim());
  },
  [input, isLoading, onGenerate],  // only recreate if these values change
);
```

**The dependency array** `[input, isLoading, onGenerate]` tells React: "only create a new version of this function when one of these values changes." This is important for performance.

**Conditional Rendering**: The button shows different content based on `isLoading`:

```typescript
{isLoading ? (
  <span className="flex items-center justify-center gap-2">
    <span className="... animate-spin" />
    Generating...
  </span>
) : (
  'Generate Component'
)}
```

### Key Tailwind Classes Used

| Class | What It Does |
|-------|-------------|
| `w-72` | Fixed width of 288px (72 x 4px) |
| `h-screen` | Full viewport height |
| `sticky top-0` | Sticks to top when scrolling |
| `flex flex-col` | Vertical flex layout |
| `flex-1` | Takes remaining space |
| `overflow-y-auto` | Scroll when content overflows |
| `bg-gray-900` | Dark background |
| `border-r border-gray-800` | Right border with subtle color |
| `bg-linear-to-r from-violet-600 to-indigo-600` | Gradient background |
| `animate-spin` | CSS rotation animation for loading spinner |
| `transition-colors` | Smooth color change on hover |
| `disabled:opacity-50` | Semi-transparent when disabled |

## Step 3: Update the App Shell

Replace `src/App.tsx` with the layout structure. For now, the center and right panels are placeholders:

```typescript
// src/App.tsx
// purpose: main app shell with state management and 3-panel layout

import { useState } from 'react';
import type { GenerationState, GalleryState } from './types';
import { Sidebar } from './prompt-input';

export const App = () => {
  const [apiKey, setApiKey] = useState(() => localStorage.getItem('openai_api_key') ?? '');
  const [generationState] = useState<GenerationState>({ status: 'idle' });
  const [galleryState] = useState<GalleryState>({ status: 'idle' });

  const handleGenerate = (prompt: string) => {
    console.log('Generate:', prompt);
    // we'll implement this in Class 3
  };

  return (
    <div className="flex h-screen bg-gray-950 text-white overflow-hidden">
      {/* left sidebar - prompt input */}
      <Sidebar
        onGenerate={handleGenerate}
        isLoading={generationState.status === 'loading'}
        apiKey={apiKey}
        onApiKeySave={setApiKey}
      />

      {/* center - preview (placeholder) */}
      <div className="flex-1 flex items-center justify-center">
        <p className="text-gray-500">Preview panel - coming in Class 4</p>
      </div>

      {/* right sidebar - variants (placeholder) */}
      <aside className="w-48 bg-gray-900 border-l border-gray-800 flex items-center justify-center">
        <p className="text-xs text-gray-500">Gallery - coming in Class 5</p>
      </aside>
    </div>
  );
};
```

### How the 3-Panel Layout Works

```
+----------+---------------------------+--------+
| Sidebar  |       Center Panel        | Gallery|
|  w-72    |        flex-1             |  w-48  |
| (288px)  |    (takes all space)      | (192px)|
|          |                           |        |
+----------+---------------------------+--------+
```

The layout uses Flexbox:

```tsx
<div className="flex h-screen overflow-hidden">
  <aside className="w-72">...</aside>    {/* fixed 288px */}
  <div className="flex-1">...</div>       {/* stretches to fill */}
  <aside className="w-48">...</aside>    {/* fixed 192px */}
</div>
```

- `flex` makes the container a flex container (children laid out horizontally)
- `h-screen` makes it fill the full viewport height
- `overflow-hidden` prevents any scrolling on the main container
- `w-72` and `w-48` give fixed widths to the sidebars
- `flex-1` makes the center panel take all remaining space

### Key Concept: useState Initializer Function

```typescript
const [apiKey, setApiKey] = useState(() => localStorage.getItem('openai_api_key') ?? '');
```

The function `() => localStorage.getItem(...)` is called a **lazy initializer**. It only runs once, on the first render. If we wrote `useState(localStorage.getItem(...))`, the `localStorage.getItem()` call would execute on every render (even though its return value would be ignored after the first render). The lazy initializer avoids that unnecessary work.

The `??` is the **nullish coalescing operator**. It returns the right side only if the left side is `null` or `undefined`. This is better than `||` because `||` also catches empty strings and `0`, which might be valid values.

## Step 4: Run and Verify

```bash
npm run dev
```

You should see:
- A dark sidebar on the left with the logo, API key input, textarea, and example prompts
- A large center area with placeholder text
- A narrow right sidebar with placeholder text

Try these interactions:
- Type in the textarea - the character count doesn't show but the text appears
- Click an example prompt - it fills the textarea
- Click "Generate Component" - check the browser console, you should see "Generate: ..." logged
- The button is disabled when the textarea is empty

---

## Key Concepts Recap

**Component Composition** - Breaking the UI into small, focused components. The `Sidebar` component handles its own form state while delegating the actual generation logic to its parent via `onGenerate` prop.

**Props** - Data flows down from parent to child. The parent `App` owns the state and passes callback functions (`onGenerate`, `onApiKeySave`) to the child. The child calls these functions to communicate back up.

**Controlled Components** - Form inputs where React state is the "single source of truth". Every change goes through `useState` and `onChange`. This gives you full control over validation, formatting, and submission.

**Flexbox Layout** - CSS `flex` creates horizontal layouts. `flex-1` fills remaining space. Fixed widths on sidebars + `flex-1` on the center panel creates a responsive 3-panel layout.

**Discriminated Unions** - TypeScript pattern that makes state management type-safe. The `status` field determines which other properties exist, and TypeScript enforces this at compile time.

---

## What's Next

In Class 3, we'll connect to the OpenAI API. We'll send the user's prompt, receive generated JSX code, clean it up with regex, and manage the generation state transitions (idle -> loading -> success/error).
