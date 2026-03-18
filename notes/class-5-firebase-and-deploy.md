# Class 5: Firebase Integration & Deployment

## What You'll Learn

- Setting up Firebase and Firestore in the Firebase Console
- Initializing the Firebase SDK with environment variables
- CRUD operations with Firestore (Create, Read, Delete)
- Querying Firestore with ordering and limits
- Building a gallery with thumbnail previews using a CSS scaling trick
- Skeleton loading states for better UX
- Deploying a Vite app to Vercel

## What You'll Build

The right sidebar gallery that displays all saved components as thumbnails, plus the Firebase backend that persists everything. By the end, you'll deploy the finished app to Vercel.

---

## Step 1: Set Up Firebase

### Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com)
2. Click "Create a project"
3. Name it (e.g., "ai-component-builder")
4. Disable Google Analytics (not needed for this project)
5. Click "Create project"

### Create a Firestore Database

1. In the Firebase Console, go to **Build > Firestore Database**
2. Click "Create database"
3. Choose "Start in test mode" (allows reads and writes without authentication - fine for a demo project)
4. Select a region close to you
5. Click "Enable"

### Get Firebase Config

1. Go to **Project Settings** (gear icon)
2. Under "Your apps", click the **web icon** (`</>`)
3. Register your app (any nickname)
4. Copy the `firebaseConfig` object. It looks like:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project-id",
  storageBucket: "your-project.firebasestorage.app",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

### Add Config to Environment Variables

Create a `.env` file in your project root (copy from `.env.example`):

```
VITE_FIREBASE_API_KEY=AIzaSy...
VITE_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-project-id
VITE_FIREBASE_STORAGE_BUCKET=your-project.firebasestorage.app
VITE_FIREBASE_MESSAGING_SENDER_ID=123456789
VITE_FIREBASE_APP_ID=1:123456789:web:abc123
```

Remember: `.env` is in `.gitignore`, so your keys won't be committed to git.

## Step 2: Create the Firebase Module

Create `src/firebase.ts`:

```typescript
// src/firebase.ts
// purpose: firebase initialization and firestore CRUD operations for saved components

import { initializeApp } from 'firebase/app';
import {
  getFirestore,
  collection,
  addDoc,
  getDocs,
  deleteDoc,
  doc,
  query,
  orderBy,
  limit,
} from 'firebase/firestore';
import type { ComponentDocument } from './types';

const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY as string,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN as string,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID as string,
  storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET as string,
  messagingSenderId: import.meta.env.VITE_FIREBASE_MESSAGING_SENDER_ID as string,
  appId: import.meta.env.VITE_FIREBASE_APP_ID as string,
};

const isConfigured = Object.values(firebaseConfig).every(Boolean);
const app = isConfigured ? initializeApp(firebaseConfig) : null;
const db = app ? getFirestore(app) : null;

const COLLECTION_NAME = 'components';

export const isFirebaseConfigured = (): boolean => isConfigured;

export const saveComponent = async (
  prompt: string,
  code: string,
  title: string,
): Promise<ComponentDocument> => {
  if (!db) throw new Error('Firebase is not configured');
  const data = { prompt, code, title, createdAt: Date.now() };
  const docRef = await addDoc(collection(db, COLLECTION_NAME), data);
  return { id: docRef.id, ...data };
};

export const listComponents = async (): Promise<ComponentDocument[]> => {
  if (!db) throw new Error('Firebase is not configured');
  const q = query(
    collection(db, COLLECTION_NAME),
    orderBy('createdAt', 'desc'),
    limit(50),
  );
  const snapshot = await getDocs(q);
  return snapshot.docs.map((d) => ({
    id: d.id,
    ...d.data(),
  })) as ComponentDocument[];
};

export const deleteComponent = async (id: string): Promise<void> => {
  if (!db) throw new Error('Firebase is not configured');
  await deleteDoc(doc(db, COLLECTION_NAME, id));
};
```

### How This Works

**Conditional Initialization:**
```typescript
const isConfigured = Object.values(firebaseConfig).every(Boolean);
const app = isConfigured ? initializeApp(firebaseConfig) : null;
```

`Object.values(firebaseConfig)` gets all config values as an array. `.every(Boolean)` checks that every value is truthy (not empty string, null, or undefined). If any env var is missing, Firebase won't initialize and `isFirebaseConfigured()` returns false. This lets the app work without Firebase during development.

**Firestore vs Realtime Database:**

Firebase has two database products:

| Feature | Firestore | Realtime Database |
|---------|-----------|-------------------|
| Data model | Collections of documents (like folders of JSON files) | One giant JSON tree |
| Queries | Complex queries with ordering, filtering, limits | Limited querying |
| Offline support | Built-in | Built-in |
| Pricing | Pay per read/write/delete | Pay per bandwidth |
| Best for | Most applications | Real-time chat, presence |

We use Firestore because its document model fits our data perfectly - each saved component is a document with `prompt`, `code`, `title`, and `createdAt` fields.

**CRUD Operations:**

**Create (saveComponent):**
```typescript
const docRef = await addDoc(collection(db, 'components'), data);
```
`addDoc` creates a new document with an auto-generated ID. `collection(db, 'components')` references the "components" collection (Firestore creates it automatically on first write).

**Read (listComponents):**
```typescript
const q = query(
  collection(db, 'components'),
  orderBy('createdAt', 'desc'),
  limit(50),
);
const snapshot = await getDocs(q);
```
`query()` builds a query with constraints. `orderBy('createdAt', 'desc')` sorts newest first. `limit(50)` caps results. `getDocs()` executes the query and returns a snapshot.

The snapshot contains `docs`, an array of document snapshots. We map over them to extract the data:
```typescript
snapshot.docs.map((d) => ({ id: d.id, ...d.data() }))
```
Each doc has `.id` (the auto-generated ID) and `.data()` (the document fields).

**Delete (deleteComponent):**
```typescript
await deleteDoc(doc(db, 'components', id));
```
`doc(db, 'components', id)` creates a reference to a specific document by its ID. `deleteDoc()` removes it.

## Step 3: Build the Gallery Sidebar

Create `src/gallery.tsx`:

```typescript
// src/gallery.tsx
// purpose: right sidebar with saved component variants as thumbnails

import { useState, useCallback, useMemo } from 'react';
import type { GalleryGridProps, ComponentDocument } from './types';
import { buildSrcdoc } from './preview-panel';

export const VariantsSidebar = ({ state, onRefresh }: GalleryGridProps) => {
  return (
    <aside className="w-48 bg-gray-900 border-l border-gray-800 flex flex-col h-screen sticky top-0">
      {/* header */}
      <div className="px-3 py-3 border-b border-gray-800 flex items-center justify-between">
        <span className="text-xs font-semibold text-gray-300">Variants</span>
        <button
          onClick={onRefresh}
          disabled={state.status === 'loading'}
          className="text-xs text-gray-500 hover:text-white disabled:opacity-50 transition-colors"
        >
          Refresh
        </button>
      </div>

      {/* variants list */}
      <div className="flex-1 overflow-y-auto p-2 space-y-2">
        {state.status === 'idle' || state.status === 'loading' ? (
          Array.from({ length: 3 }).map((_, i) => (
            <div key={i} className="aspect-[3/4] bg-gray-800 rounded-lg animate-pulse" />
          ))
        ) : state.status === 'error' ? (
          <div className="text-center py-4">
            <p className="text-xs text-red-400">{state.message}</p>
            <button
              onClick={onRefresh}
              className="mt-2 text-xs text-violet-400 hover:text-violet-300 transition-colors"
            >
              Retry
            </button>
          </div>
        ) : state.components.length === 0 ? (
          <div className="text-center py-6">
            <p className="text-xs text-gray-500">No saved variants</p>
            <p className="text-xs text-gray-600 mt-1">Generate and save components</p>
          </div>
        ) : (
          state.components.map((component) => (
            <VariantCard key={component.id} component={component} />
          ))
        )}
      </div>
    </aside>
  );
};

const VariantCard = ({ component }: { component: ComponentDocument }) => {
  const [copied, setCopied] = useState(false);
  const srcdoc = useMemo(() => buildSrcdoc(component.code), [component.code]);

  const handleCopy = useCallback(async () => {
    await navigator.clipboard.writeText(component.code);
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  }, [component.code]);

  return (
    <div className="group rounded-lg border border-gray-800 overflow-hidden hover:border-gray-600 transition-colors cursor-pointer">
      {/* thumbnail */}
      <div className="relative aspect-[3/4] overflow-hidden bg-white">
        <div
          className="absolute inset-0 origin-top-left"
          style={{ transform: 'scale(0.25)', width: '400%', height: '400%' }}
        >
          <iframe
            srcDoc={srcdoc}
            sandbox="allow-scripts"
            title={component.title}
            className="w-full h-full pointer-events-none"
            tabIndex={-1}
          />
        </div>
        {/* copy overlay on hover */}
        <div className="absolute inset-0 bg-black/0 group-hover:bg-black/40 transition-colors flex items-center justify-center">
          <button
            onClick={handleCopy}
            className="opacity-0 group-hover:opacity-100 text-xs px-2.5 py-1 bg-white text-gray-900 rounded-md font-medium transition-opacity"
          >
            {copied ? 'Copied!' : 'Copy'}
          </button>
        </div>
      </div>
      {/* label */}
      <div className="px-2 py-1.5 bg-gray-900">
        <p className="text-xs text-gray-400 truncate">{component.title}</p>
      </div>
    </div>
  );
};
```

### The Thumbnail Scaling Trick

This is one of the most clever parts of the project. How do you show a miniature preview of a component?

We can't take screenshots in the browser. Instead, we render the full component in an iframe and then scale it down with CSS:

```html
<div className="relative aspect-[3/4] overflow-hidden">  <!-- visible container -->
  <div style={{ transform: 'scale(0.25)', width: '400%', height: '400%' }}>  <!-- scaled wrapper -->
    <iframe ... />  <!-- full-size iframe -->
  </div>
</div>
```

Here's the math:

1. The outer container has a fixed size (e.g., 144px x 192px with `aspect-[3/4]`)
2. The inner div is made 4x larger: `width: 400%; height: 400%` (576px x 768px)
3. Then scaled down to 25%: `transform: scale(0.25)` brings it back to 144px x 192px
4. `origin-top-left` ensures scaling happens from the top-left corner
5. `overflow-hidden` on the parent clips anything outside the container

The result: the iframe renders at full size (576x768), then CSS visually shrinks it to fit the thumbnail card. The content looks exactly like the real component, just smaller.

**`pointer-events: none`** prevents users from accidentally interacting with the iframe (clicking links, typing in inputs, etc.).

### Skeleton Loading States

```typescript
Array.from({ length: 3 }).map((_, i) => (
  <div key={i} className="aspect-[3/4] bg-gray-800 rounded-lg animate-pulse" />
))
```

`animate-pulse` creates a fade-in/fade-out animation that signals "content is loading." Using `Array.from({ length: 3 })` creates 3 placeholder cards to approximate the final layout. This prevents layout shift when data loads.

### Tailwind Group Hover Pattern

```html
<div className="group ...">
  <div className="... group-hover:bg-black/40">
    <button className="opacity-0 group-hover:opacity-100">Copy</button>
  </div>
</div>
```

The `group` class marks a parent element. Then `group-hover:` utilities on child elements activate when the **parent** is hovered. This lets us show a "Copy" button overlay when hovering over the entire card, not just the button itself.

## Step 4: Wire Everything Together in App.tsx

Update `src/App.tsx` to its final form with Firebase integration:

```typescript
// src/App.tsx
// purpose: main app shell with state management, openai integration, and 3-panel layout

import { useState, useEffect, useCallback } from 'react';
import OpenAI from 'openai';
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

export const App = () => {
  const [apiKey, setApiKey] = useState(() => localStorage.getItem('openai_api_key') ?? '');
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

### How the Save Flow Works

1. User clicks "Save" in the PreviewPanel toolbar
2. `handleSave` is called
3. It checks `generationState.status === 'success'` (TypeScript narrows the type)
4. It checks `isFirebaseConfigured()` (graceful degradation if no Firebase)
5. Sets `isSaving = true` (shows "Saving..." on the button)
6. Calls `saveComponent()` to write to Firestore
7. Calls `fetchGallery()` to refresh the gallery sidebar
8. `finally` block sets `isSaving = false` regardless of success/failure

### How useEffect + useCallback Work Together

```typescript
const fetchGallery = useCallback(async () => {
  // ... fetch logic
}, []);

useEffect(() => {
  fetchGallery();
}, [fetchGallery]);
```

- `useCallback` with `[]` creates a stable function reference that never changes
- `useEffect` with `[fetchGallery]` runs when `fetchGallery` changes (which is only once, on mount)
- This pattern ensures the gallery is fetched exactly once when the component mounts

Why not just put the fetch logic directly in `useEffect`? Because we also call `fetchGallery()` after saving a component. By extracting it to a `useCallback`, we can reuse it in both places without duplicating code.

## Step 5: Create the Firestore Index

When you first run a `query` with `orderBy`, Firestore might require a composite index. If you see an error in the browser console with a link to create the index, click the link and Firebase will create it automatically.

Alternatively, the `createdAt` field with `orderBy('createdAt', 'desc')` usually works without a custom index since it's a single-field query.

## Step 6: Test the Complete App

1. Make sure your `.env` file has valid Firebase credentials
2. Run `npm run dev`
3. Enter your OpenAI API key
4. Generate a component
5. Click "Save" - the component should appear in the right sidebar
6. Refresh the page - saved components should load from Firestore
7. Hover over a gallery card and click "Copy" to copy the code

## Step 7: Deploy to Vercel

### Build for Production

```bash
npm run build
```

This runs TypeScript type-checking then creates an optimized production build in the `dist/` folder.

### Deploy with Vercel CLI

```bash
npm install -g vercel
vercel
```

Follow the prompts:
1. Link to your Vercel account
2. Set up the project (auto-detects Vite)
3. Deploy

### Set Environment Variables

In the Vercel dashboard for your project:
1. Go to **Settings > Environment Variables**
2. Add all your `VITE_FIREBASE_*` variables
3. Redeploy for changes to take effect

### Alternative: Deploy via GitHub

1. Push your code to GitHub
2. Go to [vercel.com](https://vercel.com) and import the repository
3. Vercel auto-detects Vite and configures the build
4. Add environment variables in project settings
5. Every push to `main` auto-deploys

---

## Key Concepts Recap

**Firestore Data Model** - Data is organized in collections (like tables) containing documents (like rows). Each document is a JSON-like object with fields. Documents can contain nested objects and arrays.

**Conditional Initialization** - `Object.values(config).every(Boolean)` checks that all config values exist before initializing Firebase. This prevents crashes when env vars are missing and allows the app to work without Firebase.

**CSS Scaling Trick** - To create thumbnails of live components: render at full size, then use `transform: scale(0.25)` with `width: 400%` to shrink visually while keeping the content readable.

**Skeleton Loading** - Placeholder UI that mimics the shape of real content with `animate-pulse`. Shows users that content is loading and prevents layout shift.

**Group Hover** - Tailwind's `group` + `group-hover:` pattern lets child elements respond to a parent's hover state. Used for overlay effects like showing a Copy button on card hover.

**try/finally** - The `finally` block runs whether the `try` succeeded or the `catch` caught an error. Perfect for cleanup like setting `isSaving = false`.

---

## Project Complete

You've built a full-stack AI Component Builder with:

- **Vite + React + TypeScript** - Modern build tooling with type safety
- **Tailwind CSS v4** - Utility-first styling with dark theme
- **OpenAI API** - AI-powered code generation from natural language
- **iframe Sandboxing** - Secure live preview of generated components
- **prism-react-renderer** - Syntax-highlighted code display
- **Firebase Firestore** - Persistent storage for saved components
- **Vercel Deployment** - Production hosting with auto-deploy

### React Concepts You've Mastered

| Concept | Where Used |
|---------|-----------|
| `useState` | Form inputs, tab state, API key, generation state |
| `useEffect` | Gallery fetch on mount |
| `useCallback` | Event handlers, API calls |
| `useMemo` | iframe srcdoc generation |
| Controlled components | Textarea, inputs |
| Conditional rendering | State-based UI (idle/loading/success/error) |
| Component composition | Small focused components with props |
| Discriminated unions | Type-safe state management |

### What to Add Next (Challenge Ideas)

- Add authentication so users have their own saved components
- Add a "Share" button that creates a public link to a component
- Support multiple AI models (GPT-4o, Claude, etc.)
- Add responsive/desktop/tablet preview size toggles
- Add component history so users can undo/redo
- Add a dark/light theme toggle in the preview iframe
