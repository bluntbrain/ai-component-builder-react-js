# Class 11: Memoization & Performance

## What You'll Learn

- What memoization means and why it matters for performance
- How React re-renders work and what triggers them
- `useMemo` for caching expensive computations
- `useCallback` for caching function references
- `React.memo` for preventing unnecessary component re-renders
- Why real-world projects (like OpenAI's Realtime Console) rely heavily on these hooks
- Common memoization mistakes and when NOT to memoize

## Overview

You won't build anything new today. Instead, you'll look at hooks that are already in your codebase and understand WHY they exist. You've been using `useCallback` and `useMemo` since Class 3. Today you learn what they actually do under the hood, and why getting them wrong can cause bugs -- not just slow code.

---

## What is Memoization?

Memoization is a technique where you cache the result of a function so you don't recompute it when the inputs haven't changed.

**Analogy**: Think of a restaurant kitchen. The chef could make garlic butter from scratch for every single order. Or they could make a big batch at the start of the shift and grab a spoonful when needed. As long as the recipe (inputs) doesn't change, the result is the same -- so why redo the work?

Here's the concept in plain JavaScript, no React involved:

```typescript
// a simple memoize function -- caches results based on the argument
function memoize(fn: (arg: number) => number) {
  const cache = new Map<number, number>();

  return (arg: number): number => {
    if (cache.has(arg)) {
      return cache.get(arg)!;
    }
    const result = fn(arg);
    cache.set(arg, result);
    return result;
  };
}

// find the nth prime number -- brute force, gets slow fast
function findNthPrime(n: number): number {
  let count = 0;
  let num = 1;
  while (count < n) {
    num++;
    let isPrime = true;
    for (let i = 2; i <= Math.sqrt(num); i++) {
      if (num % i === 0) {
        isPrime = false;
        break;
      }
    }
    if (isPrime) count++;
  }
  return num;
}

// without memoization
console.time("1st call (no cache)");
findNthPrime(100000); // ~120ms -- does all the math
console.timeEnd("1st call (no cache)");

console.time("2nd call (no cache)");
findNthPrime(100000); // ~120ms -- does all the math AGAIN
console.timeEnd("2nd call (no cache)");

// with memoization
const memoizedPrime = memoize(findNthPrime);

console.time("1st call (memoized)");
memoizedPrime(100000); // ~120ms -- first time, has to compute
console.timeEnd("1st call (memoized)");

console.time("2nd call (memoized)");
memoizedPrime(100000); // ~0.01ms -- instant cache hit
console.timeEnd("2nd call (memoized)");

console.time("3rd call (different input)");
memoizedPrime(200000); // ~280ms -- new input, has to compute
console.timeEnd("3rd call (different input)");

console.time("4th call (cached input)");
memoizedPrime(100000); // ~0.01ms -- still cached from before
console.timeEnd("4th call (cached input)");
```

The difference is dramatic: **120ms vs 0.01ms** -- that's 12,000x faster on the cached call. The first call always pays the full cost, but every repeat with the same input is essentially free.

**Key insight**: Memoization trades memory for speed. You store the previous result so you can skip the work next time. React's `useMemo` and `useCallback` are just React-specific versions of this same idea.

---

## React Re-renders -- Why They Happen

Before memoization makes sense, you need to understand re-renders. This is the most important section in these notes.

### What triggers a re-render?

1. **State changes** -- calling a `useState` setter (like `setInput('hello')`)
2. **Parent re-renders** -- when a parent component re-renders, ALL its children re-render too
3. **Context changes** -- when a context value updates, every consumer re-renders

### What happens during a re-render?

React calls your component function again, top to bottom. Every variable, every function, every object literal is **recreated from scratch**:

```typescript
const MyComponent = () => {
  // on EVERY render, these are brand new:
  const style = { padding: 16 };           // new object
  const handleClick = () => { ... };         // new function
  const filtered = items.filter(fn);         // new array

  return <Child style={style} onClick={handleClick} data={filtered} />;
};
```

### The referential equality problem

This is the core issue that memoization solves:

```typescript
// in JavaScript, two objects with identical content are NOT equal
{} === {}                     // false
[] === []                     // false
(() => {}) === (() => {})     // false

// but primitives ARE equal
'hello' === 'hello'           // true
42 === 42                     // true
```

So when your component re-renders and creates `{ padding: 16 }` again, React sees it as a **different** object than last time. This means:

- Child components see "new" props and re-render
- `useEffect` sees "new" dependencies and re-runs
- The iframe gets a "new" srcDoc string and reloads

### Try this

Open React DevTools in Chrome, go to Settings (gear icon), and enable **"Highlight updates when components render"**. Now type a character in the prompt input. Watch which components flash. Every flash is a re-render. You'll see more flashing than you expect.

---

## useMemo -- Caching Expensive Values

### How it works

```typescript
const cachedValue = useMemo(() => expensiveFunction(input), [input]);
```

1. On first render, React calls `expensiveFunction(input)` and stores the result
2. On next render, React checks: did `input` change? (using `Object.is`)
3. If **unchanged** -- returns the cached result without calling the function
4. If **changed** -- calls the function again, caches the new result

### Example 1: Building the iframe HTML

Open `src/preview-panel.tsx` and look at line 170:

```typescript
// preview-panel.tsx -- inside the WebPreview component
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

Why does this matter?

- `buildSrcdoc` constructs a complete HTML document -- 30+ lines of template literal with React, Babel, and Tailwind loaded via CDN
- **Without useMemo**: every time WebPreview re-renders (parent state change, tab switch, anything), it rebuilds the entire HTML string. Worse: because it's a new string, the iframe gets a new `srcDoc` and **reloads completely** -- white flash, scripts re-downloading, component re-mounting
- **With useMemo**: only rebuilds when `code` actually changes. Same string reference = iframe stays stable, no reload

This is a textbook useMemo case: expensive computation + the result is passed to a DOM attribute that triggers visible side effects when it changes.

### Example 2: Gallery thumbnail previews

Open `src/gallery.tsx` and look at line 55:

```typescript
// gallery.tsx -- inside the VariantCard component
const VariantCard = ({ component }: { component: ComponentDocument }) => {
  const [copied, setCopied] = useState(false);
  const srcdoc = useMemo(() => buildSrcdoc(component.code), [component.code]);
  // ...
};
```

Same pattern, but now multiply the impact: if the gallery has 10 saved components, that's 10 `VariantCard` instances. Without useMemo, every gallery re-render triggers 10 calls to `buildSrcdoc` and 10 iframe reloads. With useMemo, each card only rebuilds its srcdoc when its specific `component.code` changes.

### Try this exercise

1. In `preview-panel.tsx`, temporarily change line 170 to remove useMemo:
   ```typescript
   const srcdoc = buildSrcdoc(code);
   ```
2. Add a console.log at the top of `buildSrcdoc`:
   ```typescript
   export const buildSrcdoc = (jsxCode: string): string => {
     console.log("buildSrcdoc called");
     return `<!DOCTYPE html>...`;
   };
   ```
3. Generate a component, then switch between the Preview and Code tabs
4. Check the console -- how many times does `buildSrcdoc` run?
5. Put useMemo back and repeat. Notice the difference.

---

## useCallback -- Caching Function References

### How it works

```typescript
const cachedFunction = useCallback(
  (args) => {
    // function body
  },
  [dep1, dep2],
);
```

`useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`. The difference:

|              | useMemo                                   | useCallback                                         |
| ------------ | ----------------------------------------- | --------------------------------------------------- |
| **Caches**   | The return VALUE of a function            | The function ITSELF                                 |
| **Returns**  | Any value (string, number, object, array) | A function                                          |
| **Use when** | Expensive computation                     | Passing function as prop or as useEffect dependency |

useCallback doesn't make the function run faster. It returns the **same function reference** between renders, so downstream consumers (child components, effects) don't see a "new" function every time.

### Example 1: fetchGallery -- preventing infinite useEffect loops

Open `src/App.tsx` and look at lines 36-50:

```typescript
// App.tsx
const fetchGallery = useCallback(async () => {
  if (!isFirebaseConfigured()) return;
  setGalleryState({ status: "loading" });
  try {
    const components = await listComponents();
    setGalleryState({ status: "success", components });
  } catch (err) {
    const message =
      err instanceof Error ? err.message : "Failed to load gallery";
    setGalleryState({ status: "error", message });
  }
}, []);

useEffect(() => {
  fetchGallery();
}, [fetchGallery]);
```

**Why `[]` dependencies?** This function doesn't read any values from the component scope that change -- `isFirebaseConfigured`, `listComponents`, and `setGalleryState` are all stable references. So the function never needs to be recreated.

**Why it matters**: `fetchGallery` is used as a dependency of `useEffect`. If fetchGallery got a new reference every render:

1. Component renders -> fetchGallery is a new function
2. useEffect sees new dependency -> runs fetchGallery
3. fetchGallery calls setGalleryState -> triggers re-render
4. Go back to step 1 -- **infinite loop**

With useCallback and `[]`, fetchGallery keeps the same reference forever, so the effect only runs once on mount.

`fetchGallery` is also passed to `VariantsSidebar` as the `onRefresh` prop. Stable reference means the sidebar doesn't see "new props" every render.

### Example 2: handleGenerate -- why apiKey is in the dependency array

Look at `src/App.tsx` line 52:

```typescript
// App.tsx
const handleGenerate = useCallback(
  async (prompt: string) => {
    if (!apiKey) return;
    setGenerationState({ status: "loading" });
    try {
      const openai = new OpenAI({
        apiKey, // <-- reads apiKey from closure
        dangerouslyAllowBrowser: true,
      });
      const response = await openai.chat.completions.create({
        model: "gpt-4o",
        messages: [
          { role: "system", content: "Return only raw JSX..." },
          { role: "user", content: prompt },
        ],
        temperature: 0.7,
        max_tokens: 2000,
      });
      const raw = response.choices[0]?.message?.content ?? "";
      const code = cleanGeneratedCode(raw);
      if (!code) {
        setGenerationState({
          status: "error",
          message: "No code was generated.",
        });
        return;
      }
      setGenerationState({ status: "success", code, prompt });
    } catch (err) {
      const message = err instanceof Error ? err.message : "Generation failed";
      setGenerationState({ status: "error", message });
    }
  },
  [apiKey],
);
```

**Why `[apiKey]`?** This function reads `apiKey` from the closure to create the OpenAI client. When the user enters a new API key:

1. `apiKey` state changes -> component re-renders
2. useCallback sees `apiKey` changed -> creates a new function that captures the new key
3. Next time the user clicks "Generate," the function uses the updated key

**What if you forgot `apiKey` in the deps?** The function would always use the stale initial API key. The user would change their key, click Generate, and it would still use the old key. This is called a **stale closure bug** -- one of the hardest bugs to track down in React.

This function is passed to `Sidebar` as the `onGenerate` prop.

### Example 3: handleSave -- multiple dependencies

Look at `src/App.tsx` line 86:

```typescript
// App.tsx
const handleSave = useCallback(async () => {
  if (generationState.status !== "success") return;
  if (!isFirebaseConfigured()) return;
  setIsSaving(true);
  try {
    const title = extractTitle(generationState.prompt);
    await saveComponent(generationState.prompt, generationState.code, title);
    await fetchGallery();
  } catch (err) {
    console.error("failed to save component:", err);
  } finally {
    setIsSaving(false);
  }
}, [generationState, fetchGallery]);
```

- Depends on `generationState` (reads `.status`, `.prompt`, `.code`) and `fetchGallery` (calls it)
- When the user generates new code, `generationState` changes, so `handleSave` gets a new reference that captures the latest generated code
- `fetchGallery` has `[]` deps so it never changes -- but including it is correct because ESLint's exhaustive-deps rule requires listing every value read from the closure

### Example 4: handleSubmit -- form handler with multiple inputs

Open `src/prompt-input.tsx` line 30:

```typescript
// prompt-input.tsx
const handleSubmit = useCallback(
  (e: FormEvent) => {
    e.preventDefault();
    if (!input.trim() || isLoading) return;
    onGenerate(input.trim());
  },
  [input, isLoading, onGenerate],
);
```

- Reads `input` (local state), `isLoading` (prop), and calls `onGenerate` (prop)
- Every keystroke changes `input`, so this function updates every keystroke -- that's fine for a form handler
- The important thing is that it always has the current values when the user hits Enter

### Example 5: handleCopy -- note what's NOT in the deps

Open `src/preview-panel.tsx` line 206:

```typescript
// preview-panel.tsx -- inside CopyButton
const handleCopy = useCallback(async () => {
  await navigator.clipboard.writeText(code);
  setCopied(true);
  setTimeout(() => setCopied(false), 2000);
}, [code]);
```

Notice: `setCopied` is NOT in the dependency array. Why? React **guarantees** that state setter functions from `useState` have a stable identity -- they never change between renders. So you don't need to list them. Same applies to `dispatch` from `useReducer`.

---

## React.memo -- Wrapping Components

### How it works

```typescript
import { memo } from 'react';

const MyComponent = memo(({ name, onClick }: Props) => {
  return <button onClick={onClick}>{name}</button>;
});
```

`React.memo` wraps a component to skip re-rendering when its props haven't changed. It does a **shallow comparison** of each prop using `Object.is`.

- Props the same? -> skip render, reuse previous output
- Any prop different? -> re-render normally

### Our codebase doesn't use React.memo (yet)

The AI Component Builder is small enough that unnecessary re-renders don't cause visible performance issues. But let's look at where it COULD help -- and where it WOULDN'T.

### Where it would help: VariantCard

```typescript
// gallery.tsx -- current code
const VariantCard = ({ component }: { component: ComponentDocument }) => {
  const [copied, setCopied] = useState(false);
  const srcdoc = useMemo(() => buildSrcdoc(component.code), [component.code]);
  // ... renders iframe thumbnail and copy button
};

// with React.memo
const VariantCard = memo(({ component }: { component: ComponentDocument }) => {
  const [copied, setCopied] = useState(false);
  const srcdoc = useMemo(() => buildSrcdoc(component.code), [component.code]);
  // ...
});
```

When the gallery re-renders (say, after saving a new component), ALL VariantCards re-render. With `React.memo`, only the card whose `component` prop reference changed would re-render. The other 9 cards skip rendering entirely.

This works because each `component` object from Firestore has a stable reference -- it doesn't get recreated unless the data changes.

### Where it would NOT help: TabButton

```typescript
// preview-panel.tsx -- current code
const TabButton = ({ active, onClick, children }: {
  active: boolean;
  onClick: () => void;
  children: React.ReactNode;
}) => (
  <button onClick={onClick} className={/* ... */}>
    {children}
  </button>
);
```

You might think wrapping this in `memo` would help. But look at how it's used:

```typescript
<TabButton active={activeTab === 'preview'} onClick={() => setActiveTab('preview')}>
  Preview
</TabButton>
```

That `onClick={() => setActiveTab('preview')}` is an **inline arrow function** -- it creates a new function reference every render. So even with `React.memo`, the component would always see a "new" `onClick` prop and re-render anyway.

To make `memo` work here, you'd also need to wrap the onClick in useCallback:

```typescript
const handlePreviewTab = useCallback(() => setActiveTab('preview'), []);
const handleCodeTab = useCallback(() => setActiveTab('code'), []);

<TabButton active={activeTab === 'preview'} onClick={handlePreviewTab}>
  Preview
</TabButton>
```

But TabButton is tiny -- just a `<button>` with a className. The overhead of comparing props probably costs more than just re-rendering it. **Not worth it.**

### When is React.memo worth it?

A component is a good candidate for `memo` when ALL three conditions are true:

1. It **renders often** (sits inside a parent that updates frequently)
2. It receives **stable props** (or you're willing to memoize them)
3. It's **expensive to render** (complex JSX, many children, or computations)

For small leaf components like buttons and labels, just let them re-render.

---

## Why OpenAI's Realtime Console Uses useCallback

Let's zoom out from our project. OpenAI's `openai-realtime-console` on GitHub is a React app that connects to the Realtime API via WebSocket for live audio and text streaming. It uses `useCallback` extensively -- and for a different reason than performance.

### The pattern

```typescript
// conceptual example from a realtime app (simplified)
const handleMessage = useCallback(
  (event: MessageEvent) => {
    // process incoming audio or text chunk
    appendToTranscript(event.data);
  },
  [appendToTranscript],
);

useEffect(() => {
  websocket.addEventListener("message", handleMessage);
  return () => websocket.removeEventListener("message", handleMessage);
}, [websocket, handleMessage]);
```

### Why it matters for correctness (not just performance)

1. **WebSocket event handlers must have stable references.** When useEffect re-runs, it calls the cleanup function (removeEventListener) before adding the new listener. If `handleMessage` changed every render, the effect would remove and re-add the listener every render.

2. **During the gap between remove and re-add, messages can be dropped.** In a realtime audio stream, that means choppy audio or missing words.

3. **Streaming data arrives continuously.** Re-renders during streaming are frequent (updating the transcript, showing audio levels). Unstable callbacks would cause a re-subscription storm -- removing and adding listeners dozens of times per second.

4. **With useCallback**, the handler keeps the same reference. The effect's dependency doesn't change, so the listener stays registered. No gaps, no dropped messages.

### The takeaway

|                             | Our AI Component Builder   | OpenAI Realtime Console             |
| --------------------------- | -------------------------- | ----------------------------------- |
| **Why memoize**             | Performance optimization   | Correctness requirement             |
| **What happens without it** | Slightly slower renders    | Dropped audio, broken transcription |
| **Risk level**              | Minor visual jank at worst | Actual bugs and broken features     |

In our app, if you removed all useCallback and useMemo hooks, it would still work -- just a bit slower with iframe flicker. In a realtime app, removing them breaks the product.

---

## Common Mistakes

### Mistake 1: Memoizing everything

```typescript
// unnecessary -- simple multiplication is faster than the memoization overhead
const total = useMemo(() => price * quantity, [price, quantity]);

// just compute it directly
const total = price * quantity;
```

useMemo itself has a cost: it stores the previous value, compares dependencies with `Object.is`, and manages an internal cache. For simple operations, this overhead exceeds the computation you're trying to avoid.

### Mistake 2: Missing dependencies (stale closure)

```typescript
// bug: apiKey will always be the initial value
const handleGenerate = useCallback(async (prompt: string) => {
  const openai = new OpenAI({ apiKey }); // reads apiKey from closure
}, []); // apiKey is missing!

// fix: include apiKey
const handleGenerate = useCallback(
  async (prompt: string) => {
    const openai = new OpenAI({ apiKey });
  },
  [apiKey],
); // now the function updates when apiKey changes
```

The ESLint plugin `react-hooks/exhaustive-deps` catches this. Never disable that rule.

### Mistake 3: Inline objects defeating React.memo

```typescript
// the style object is new every render, so memo sees "new" props
<MemoizedChild style={{ padding: 16 }} />

// fix: memoize the object
const style = useMemo(() => ({ padding: 16 }), []);
<MemoizedChild style={style} />

// or if it's truly static, define it outside the component
const STYLE = { padding: 16 };
const MyComponent = () => <MemoizedChild style={STYLE} />;
```

### Mistake 4: Premature optimization

The React docs say it clearly: "Don't memoize everything. Start without memoization, measure with React DevTools Profiler, then optimize the specific bottlenecks."

**Rule of thumb**: If a component renders in under 1ms (check in React DevTools Profiler), memoization adds complexity without measurable benefit. Your time is better spent elsewhere.

---

## Quick Reference

### When to use what

| Hook          | What It Caches          | When to Use                                                 | Dependency Array                           |
| ------------- | ----------------------- | ----------------------------------------------------------- | ------------------------------------------ |
| `useMemo`     | A computed value        | Expensive calculations, objects/arrays passed as props      | Values the computation reads               |
| `useCallback` | A function reference    | Functions passed as props, functions used in useEffect deps | Values the function reads from its closure |
| `React.memo`  | Component render output | Components that re-render often with unchanged props        | N/A (compares props automatically)         |

### Decision flowchart

1. Is a computation expensive (HTML string building, filtering large arrays, complex math)?
   -> **useMemo**

2. Is a function passed as a prop to a child component, or used as a useEffect dependency?
   -> **useCallback**

3. Does a child component re-render often with the same props, and is it expensive to render?
   -> **React.memo** on the child (and make sure its props are memoized too)

4. None of the above?
   -> **Don't memoize.** Plain variables and functions are fine.

### Where we use them in our codebase

| File                | Hook        | Line | What it does                                                  |
| ------------------- | ----------- | ---- | ------------------------------------------------------------- |
| `App.tsx`           | useCallback | 36   | `fetchGallery` -- stable ref for useEffect and onRefresh prop |
| `App.tsx`           | useCallback | 52   | `handleGenerate` -- captures current apiKey                   |
| `App.tsx`           | useCallback | 86   | `handleSave` -- captures current generationState              |
| `prompt-input.tsx`  | useCallback | 30   | `handleSubmit` -- form handler with current input             |
| `prompt-input.tsx`  | useCallback | 39   | `handleChipClick` -- example chip click handler               |
| `preview-panel.tsx` | useMemo     | 170  | `srcdoc` -- avoids rebuilding iframe HTML                     |
| `preview-panel.tsx` | useCallback | 206  | `handleCopy` -- copy button handler                           |
| `gallery.tsx`       | useMemo     | 55   | `srcdoc` -- avoids rebuilding thumbnail HTML                  |
| `gallery.tsx`       | useCallback | 57   | `handleCopy` -- gallery card copy handler                     |

---

## Key Concepts Recap

- **Memoization** -- caching the result of a computation so you don't redo the work when inputs haven't changed. A general CS concept that React implements with specific hooks.

- **useMemo** -- caches a computed value. Only recomputes when dependencies change. Use for expensive operations like building HTML strings or filtering large datasets.

- **useCallback** -- caches a function reference. The function itself doesn't run faster, but having a stable reference prevents child components and effects from doing unnecessary work.

- **React.memo** -- a higher-order component that skips re-rendering when props are shallowly equal. Most effective when combined with useMemo and useCallback to ensure props have stable references.

- **Dependency array** -- the second argument to useMemo and useCallback. React uses it to decide when to invalidate the cache. Every value from the component scope that the memoized code reads must be listed.

- **Referential equality** -- `{} !== {}` in JavaScript. Two objects with identical content are different references. This is why React re-renders children when you pass inline objects or functions as props.

- **Stale closure** -- a bug where a memoized function captures an old value because its dependency array is missing an entry. The function "remembers" the value from when it was created, not the current value.

---

## What's Next

This is the last concept class before you start building your own projects (Class 10). As you build, use the React DevTools Profiler to find real performance issues, and apply memoization only where it makes a measurable difference. The best code is the simplest code that works well enough.
