# Class 4: Live Preview & Code Display

## What You'll Learn

- How iframe sandboxing works and why it's essential for security
- Building an HTML document string (srcdoc) that renders React components
- How Babel standalone transpiles JSX in the browser
- Syntax highlighting with prism-react-renderer
- The Clipboard API for copy-to-clipboard functionality
- Using `useMemo` for expensive computations
- Tab-based UI with conditional rendering

## What You'll Build

The center panel of the app. When code is generated, users see two tabs:
- **Preview**: The generated component rendered live inside a phone-style iframe
- **Code**: Syntax-highlighted JSX with line numbers and a copy button

---

## Step 1: Understanding the Problem

We have a string of JSX code from OpenAI:

```jsx
<div className="bg-gray-900 p-6 rounded-xl">
  <h2 className="text-2xl font-bold text-white">Pro Plan</h2>
  <p className="text-4xl font-bold text-white mt-4">$29/mo</p>
</div>
```

We need to turn this string into a live, rendered UI component. But we can't just use `dangerouslySetInnerHTML` because:

1. The code contains JSX (not HTML) - `className` isn't valid HTML
2. It might use Tailwind classes that aren't in our app's CSS
3. **Running untrusted code in the main page is a security risk** - it could access localStorage, cookies, or call APIs with the user's credentials

The solution: render it inside a **sandboxed iframe**.

## Step 2: Build the srcdoc Template

The key to our approach is the `buildSrcdoc` function. It constructs a complete HTML document as a string, which gets loaded into an iframe via the `srcDoc` attribute.

Create `src/preview-panel.tsx`:

```typescript
// src/preview-panel.tsx
// purpose: live preview iframe and syntax-highlighted code display for generated components

import { useState, useCallback, useMemo } from 'react';
import { Highlight, themes } from 'prism-react-renderer';
import type { PreviewPanelProps } from './types';

// builds a sandboxed html document that renders jsx with react 18, babel, and tailwind via cdn
export const buildSrcdoc = (jsxCode: string): string => `
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <script src="https://unpkg.com/react@18/umd/react.production.min.js"><\/script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"><\/script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"><\/script>
  <script src="https://cdn.tailwindcss.com"><\/script>
  <style>
    body { margin: 0; padding: 16px; font-family: system-ui, -apple-system, sans-serif; background: white; }
    .error-display { color: #ef4444; padding: 16px; font-family: monospace; font-size: 14px; white-space: pre-wrap; }
  </style>
</head>
<body>
  <div id="root"></div>
  <script type="text/babel">
    try {
      const Component = () => (
        ${jsxCode}
      );
      ReactDOM.createRoot(document.getElementById('root')).render(React.createElement(Component));
    } catch (err) {
      document.getElementById('root').innerHTML = '<div class="error-display">Render error: ' + err.message + '</div>';
    }
  <\/script>
  <script>
    window.onerror = function(msg) {
      document.getElementById('root').innerHTML = '<div class="error-display">Error: ' + msg + '</div>';
    };
  <\/script>
</body>
</html>`;
```

### How This Works - Line by Line

**CDN Scripts:**
```html
<script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
```
These load React 18 as UMD (Universal Module Definition) builds. UMD scripts create global variables (`React` and `ReactDOM`) that can be used in regular `<script>` tags. We use React 18 instead of 19 because React 18's UMD builds reliably expose `ReactDOM` as a global variable.

**Babel Standalone:**
```html
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
```
Babel is normally a build-time tool that converts JSX into JavaScript. The "standalone" version runs in the browser. Any `<script type="text/babel">` block will be automatically transpiled before execution.

So this JSX:
```jsx
const Component = () => (<div className="p-4">Hello</div>);
```

Gets transpiled to:
```javascript
const Component = () => React.createElement("div", { className: "p-4" }, "Hello");
```

**Tailwind CDN:**
```html
<script src="https://cdn.tailwindcss.com"></script>
```
This loads a runtime version of Tailwind that scans the page for class names and generates CSS on the fly. It's not meant for production (too slow), but perfect for our preview iframe.

**Component Rendering:**
```javascript
const Component = () => (
  ${jsxCode}    // the AI-generated JSX gets inserted here
);
ReactDOM.createRoot(document.getElementById('root')).render(React.createElement(Component));
```

We wrap the JSX in an arrow function to create a valid React component, then render it.

**Error Handling:**
```javascript
try { ... } catch (err) {
  document.getElementById('root').innerHTML = '<div class="error-display">Render error: ' + err.message + '</div>';
}
```
If the generated code has syntax errors, Babel will throw during transpilation. The try/catch catches it and shows a friendly error message instead of a blank screen.

The `window.onerror` handler catches runtime errors that happen after the component renders (like referencing an undefined variable).

**Script Escaping:**
```
<\/script>
```
Inside a template literal that contains HTML, we escape closing script tags as `<\/script>`. Without the backslash, the browser's HTML parser would think the outer script tag is ending, breaking the page.

## Step 3: Build the Preview Panel Component

Add the rest of the components to `src/preview-panel.tsx`:

```typescript
export const PreviewPanel = ({ state, onSave, isSaving }: PreviewPanelProps) => {
  const [activeTab, setActiveTab] = useState<'preview' | 'code'>('preview');

  return (
    <div className="flex-1 flex flex-col min-h-0">
      {/* top toolbar */}
      <div className="flex items-center justify-between px-4 py-2 bg-gray-900 border-b border-gray-800">
        <div className="flex items-center gap-3">
          <div className="flex gap-1">
            <TabButton active={activeTab === 'preview'} onClick={() => setActiveTab('preview')}>
              Preview
            </TabButton>
            <TabButton active={activeTab === 'code'} onClick={() => setActiveTab('code')}>
              Code
            </TabButton>
          </div>
          {state.status === 'success' && (
            <span className="text-xs text-gray-500 border-l border-gray-700 pl-3">
              Live render
            </span>
          )}
        </div>
        <div className="flex items-center gap-2">
          {state.status === 'success' && (
            <>
              <CopyButton code={state.code} />
              <button
                onClick={onSave}
                disabled={isSaving}
                className="text-xs px-3 py-1.5 bg-emerald-600 text-white rounded-lg hover:bg-emerald-700 disabled:opacity-50 transition-colors"
              >
                {isSaving ? 'Saving...' : 'Save'}
              </button>
            </>
          )}
        </div>
      </div>

      {/* content area */}
      <div className="flex-1 flex items-center justify-center bg-gray-950/50 p-6 overflow-auto">
        {state.status === 'idle' && <IdlePlaceholder />}
        {state.status === 'loading' && <LoadingState />}
        {state.status === 'error' && <ErrorState message={state.message} />}
        {state.status === 'success' && (
          activeTab === 'preview' ? (
            <WebPreview code={state.code} />
          ) : (
            <CodeBlock code={state.code} />
          )
        )}
      </div>
    </div>
  );
};
```

### How Conditional Rendering Works

Notice how we render different content for each state:

```typescript
{state.status === 'idle' && <IdlePlaceholder />}
{state.status === 'loading' && <LoadingState />}
{state.status === 'error' && <ErrorState message={state.message} />}
{state.status === 'success' && (
  activeTab === 'preview' ? <WebPreview code={state.code} /> : <CodeBlock code={state.code} />
)}
```

This uses **short-circuit evaluation**: `condition && <Component />` renders the component only when the condition is true. Because `status` is a discriminated union, only one condition can be true at a time.

Inside the `success` branch, we can safely access `state.code` and `state.message` because TypeScript knows the status is `'success'`.

## Step 4: Build the Sub-Components

Add these below the `PreviewPanel`:

```typescript
const IdlePlaceholder = () => (
  <div className="text-center">
    <svg className="w-16 h-16 mx-auto mb-4 text-gray-700" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={1} d="M9.75 17L9 20l-1 1h8l-1-1-.75-3M3 13h18M5 17h14a2 2 0 002-2V5a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z" />
    </svg>
    <p className="text-sm text-gray-500">Describe a component to see a live preview</p>
    <p className="text-xs text-gray-600 mt-1">Your generated UI will appear here</p>
  </div>
);

const LoadingState = () => (
  <div className="flex flex-col items-center gap-4">
    <div className="w-10 h-10 border-2 border-violet-500 border-t-transparent rounded-full animate-spin" />
    <div className="text-center">
      <p className="text-sm text-gray-300">Generating component...</p>
      <p className="text-xs text-gray-500 mt-1">This usually takes a few seconds</p>
    </div>
  </div>
);

const ErrorState = ({ message }: { message: string }) => (
  <div className="text-center max-w-sm">
    <div className="w-10 h-10 mx-auto mb-3 rounded-full bg-red-500/10 flex items-center justify-center">
      <svg className="w-5 h-5 text-red-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
      </svg>
    </div>
    <p className="text-sm text-red-400">{message}</p>
    <p className="text-xs text-gray-500 mt-2">Try a different prompt or check your API key</p>
  </div>
);
```

### The Loading Spinner (CSS-only)

```html
<div className="w-10 h-10 border-2 border-violet-500 border-t-transparent rounded-full animate-spin" />
```

This creates a spinning circle with pure CSS:
- `border-2 border-violet-500` - a violet circle border
- `border-t-transparent` - the top segment is invisible, creating the "gap"
- `rounded-full` - makes it a circle
- `animate-spin` - Tailwind's built-in infinite rotation animation

No SVG or image needed.

## Step 5: The Web Preview (iframe)

```typescript
// full-width web preview
const WebPreview = ({ code }: { code: string }) => {
  const srcdoc = useMemo(() => buildSrcdoc(code), [code]);

  return (
    <div className="w-full h-full rounded-xl overflow-hidden border border-gray-800 bg-white">
      <iframe
        srcDoc={srcdoc}
        sandbox="allow-scripts"
        title="Component Preview"
        className="w-full h-full border-0"
      />
    </div>
  );
};
```

### Key Concept: useMemo

```typescript
const srcdoc = useMemo(() => buildSrcdoc(code), [code]);
```

`useMemo` caches the result of an expensive computation. `buildSrcdoc` builds a large HTML string with template literals - we don't want to rebuild it on every render. `useMemo` only recomputes when `code` changes (the dependency array).

Without `useMemo`:
- Every render calls `buildSrcdoc(code)`, creating a new string
- The iframe gets a new `srcDoc` value and reloads completely

With `useMemo`:
- `buildSrcdoc` only runs when `code` actually changes
- Same string reference = iframe doesn't reload

### Key Concept: iframe sandbox Attribute

```html
<iframe sandbox="allow-scripts" />
```

The `sandbox` attribute restricts what the iframe can do:

| What's Blocked | Why It Matters |
|---------------|----------------|
| `allow-same-origin` (NOT included) | Iframe can't access parent page's cookies, localStorage, or DOM |
| Form submission | Generated code can't submit forms |
| Popups/modals | Can't open new windows |
| Navigation | Can't redirect the parent page |
| Downloads | Can't trigger file downloads |

We only allow `allow-scripts` because the generated code needs JavaScript to run React. Everything else is blocked.

## Step 6: Syntax Highlighting with prism-react-renderer

```typescript
const CodeBlock = ({ code }: { code: string }) => (
  <div className="w-full max-w-2xl max-h-full overflow-auto rounded-xl border border-gray-800">
    <Highlight theme={themes.nightOwl} code={code} language="jsx">
      {({ style, tokens, getLineProps, getTokenProps }) => (
        <pre style={{ ...style, margin: 0, padding: '16px', fontSize: '13px' }}>
          {tokens.map((line, i) => (
            <div key={i} {...getLineProps({ line })}>
              <span className="inline-block w-8 text-right mr-4 text-gray-600 select-none">
                {i + 1}
              </span>
              {line.map((token, key) => (
                <span key={key} {...getTokenProps({ token })} />
              ))}
            </div>
          ))}
        </pre>
      )}
    </Highlight>
  </div>
);
```

### How prism-react-renderer Works

The `Highlight` component uses a **render props** pattern. Instead of rendering its own UI, it calls a function (your function) and passes it the data:

- `style` - background color and base styles from the theme
- `tokens` - the code split into syntax tokens (keywords, strings, brackets, etc.)
- `getLineProps` - generates props for each line `<div>`
- `getTokenProps` - generates props for each token `<span>` (includes color from theme)

Each token gets a different color based on its type:
- Keywords (`const`, `return`) - one color
- Strings (`"hello"`) - another color
- JSX tags (`<div>`) - another color
- etc.

The `nightOwl` theme provides a dark color scheme that matches our app.

**Line numbers** are added manually with a `<span>` showing `{i + 1}`. The `select-none` class prevents them from being selected when copying code.

## Step 7: Copy Button and Tab Button

```typescript
const CopyButton = ({ code }: { code: string }) => {
  const [copied, setCopied] = useState(false);

  const handleCopy = useCallback(async () => {
    await navigator.clipboard.writeText(code);
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  }, [code]);

  return (
    <button
      onClick={handleCopy}
      className="text-xs px-3 py-1.5 bg-gray-800 text-gray-300 rounded-lg border border-gray-700 hover:text-white hover:border-gray-600 transition-colors"
    >
      {copied ? 'Copied!' : 'Copy Code'}
    </button>
  );
};

const TabButton = ({
  active,
  onClick,
  children,
}: {
  active: boolean;
  onClick: () => void;
  children: React.ReactNode;
}) => (
  <button
    onClick={onClick}
    className={`px-3 py-1.5 text-xs font-medium rounded-lg transition-colors ${
      active
        ? 'bg-gray-800 text-white'
        : 'text-gray-500 hover:text-gray-300'
    }`}
  >
    {children}
  </button>
);
```

### Key Concept: Clipboard API

```typescript
await navigator.clipboard.writeText(code);
```

The modern Clipboard API is promise-based and requires HTTPS (or localhost). It replaces the old `document.execCommand('copy')` approach. The `await` ensures we don't show "Copied!" until the copy actually succeeds.

The feedback pattern:
1. User clicks Copy
2. Text is copied to clipboard
3. Button text changes to "Copied!"
4. After 2 seconds, `setTimeout` resets it back to "Copy Code"

## Step 8: Wire It Up in App.tsx

Update your `App.tsx` to import and use `PreviewPanel`. Replace the center placeholder:

```typescript
import { PreviewPanel } from './preview-panel';

// ... inside the return, replace the center div with:
<PreviewPanel
  state={generationState}
  onSave={() => console.log('Save - coming in Class 5')}
  isSaving={false}
/>
```

## Step 9: Test It

1. Run `npm run dev`
2. Enter your API key and a prompt
3. Click Generate
4. Switch between Preview and Code tabs
5. Click "Copy Code" and paste somewhere to verify
6. Try generating a component with invalid code to see error handling

---

## Key Concepts Recap

**iframe Sandboxing** - Isolates untrusted code in a secure container. The `sandbox` attribute restricts capabilities. Only `allow-scripts` is enabled for our use case.

**srcdoc** - Loads HTML directly into an iframe from a string, instead of from a URL. This avoids CORS issues and keeps everything self-contained.

**Babel Standalone** - A browser version of Babel that transpiles JSX to JavaScript at runtime. Used via `<script type="text/babel">`. Not suitable for production apps, but perfect for sandboxed previews.

**useMemo** - Caches expensive computations. Only recomputes when dependencies change. Use it when a calculation is costly and its inputs don't change every render.

**Clipboard API** - `navigator.clipboard.writeText()` copies text to clipboard. It's async and only works on HTTPS or localhost.

**Render Props** - A pattern where a component calls a function prop to determine what to render. prism-react-renderer uses this to give you full control over the syntax-highlighted output.

---

## What's Next

In Class 5, we'll add Firebase to save generated components, build the gallery sidebar with thumbnail previews (using a clever CSS scaling trick), and deploy the app to Vercel.
