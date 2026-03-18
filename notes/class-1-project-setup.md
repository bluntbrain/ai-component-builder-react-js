# Class 1: Project Setup - Vite, React, TypeScript & Tailwind v4

## What You'll Learn

- What Vite is and why modern React projects use it instead of Create React App
- How to scaffold a React + TypeScript project from scratch
- How Tailwind CSS v4 works (and what changed from v3)
- How environment variables work in Vite
- The purpose of every config file in the project

## What You'll Build

By the end of this class, you'll have a fully configured project skeleton that compiles, runs a dev server with hot module replacement, and renders a basic "Hello World" page with Tailwind styling. This is the foundation for everything we build in the next 4 classes.

## Final Project Structure

```
ai-component-builder/
  .gitignore
  .env.example
  index.html
  package.json
  tsconfig.json
  vite.config.ts
  src/
    main.tsx
    App.tsx
    index.css
    vite-env.d.ts
```

---

## Step 1: Scaffold the Project

Open your terminal and run:

```bash
mkdir ai-component-builder
cd ai-component-builder
git init
npm create vite@latest . -- --template react-ts
```

This uses Vite's scaffolding tool to generate a React + TypeScript project. Let's break down what happens:

- `npm create vite@latest` runs the `create-vite` package
- `.` means "scaffold in the current directory" (not a subdirectory)
- `--template react-ts` selects React with TypeScript

### Why Vite Instead of Create React App?

Create React App (CRA) uses Webpack under the hood. Vite uses a completely different approach:

- **Development**: Vite serves files using native ES modules. Your browser loads each file individually, so Vite doesn't need to bundle anything during development. This makes the dev server start in milliseconds, not seconds.
- **Production**: Vite uses Rolldown (based on Rollup) to create optimized bundles for deployment.
- **Hot Module Replacement (HMR)**: When you save a file, only that specific module is replaced in the browser. No full page reload needed.

CRA was deprecated in 2023. Vite is now the standard recommendation from the React team.

## Step 2: Install Dependencies

```bash
npm install
```

This installs the base dependencies from the scaffold. Now install our project-specific dependencies:

```bash
npm install firebase openai prism-react-renderer
npm install -D tailwindcss @tailwindcss/vite
```

What each package does:

| Package | Purpose |
|---------|---------|
| `firebase` | Database for saving generated components |
| `openai` | Official SDK to call the OpenAI API |
| `prism-react-renderer` | Syntax highlighting for code display |
| `tailwindcss` | Utility-first CSS framework (v4) |
| `@tailwindcss/vite` | Vite plugin for Tailwind v4 |

## Step 3: Clean Up Scaffold Files

The Vite scaffold creates some files we don't need. Delete them:

```bash
rm src/App.css
rm -rf src/assets
rm public/vite.svg
```

We'll replace the content of the remaining files with our own code.

## Step 4: Configure Vite

Replace the contents of `vite.config.ts`:

```typescript
// vite.config.ts
// purpose: vite configuration with react and tailwind v4 plugins
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

### How It Works

- `defineConfig()` provides TypeScript autocompletion for the config object
- `react()` enables JSX transformation and React Fast Refresh (HMR for React components)
- `tailwindcss()` processes Tailwind CSS classes at build time. In Tailwind v4, this replaces the old PostCSS plugin approach

## Step 5: Configure TypeScript

The scaffold creates 3 TypeScript config files (`tsconfig.json`, `tsconfig.app.json`, `tsconfig.node.json`). We'll consolidate into a single file for simplicity.

Delete the extra files:

```bash
rm tsconfig.app.json tsconfig.node.json
```

Replace `tsconfig.json` with:

```json
{
  "compilerOptions": {
    "target": "ES2023",
    "useDefineForClassFields": true,
    "lib": ["ES2023", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "moduleDetection": "force",
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "erasableSyntaxOnly": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedSideEffectImports": true
  },
  "include": ["src", "vite.config.ts"]
}
```

### Key Settings Explained

| Setting | What It Does |
|---------|-------------|
| `target: "ES2023"` | Compile to modern JavaScript (no need to support IE) |
| `jsx: "react-jsx"` | Uses the new JSX transform - you don't need `import React` in every file |
| `strict: true` | Enables all strict type checking. Catches more bugs at compile time |
| `moduleResolution: "bundler"` | Tells TypeScript that Vite (the bundler) will resolve imports |
| `noEmit: true` | TypeScript only checks types, doesn't produce JavaScript files. Vite handles the actual compilation |
| `noUnusedLocals: true` | Error if you declare a variable but never use it |
| `verbatimModuleSyntax: true` | Forces you to use `import type` for type-only imports |

## Step 6: Set Up the HTML Entry Point

Replace `index.html`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet" />
    <title>AI Component Builder</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

### How It Works

- **No bundled JS in HTML**: Vite serves `src/main.tsx` directly as an ES module during development. In production, it gets bundled automatically.
- **Google Fonts**: We preload the Inter font family. `preconnect` tells the browser to start the connection early, making font loading faster.
- **`<div id="root">`**: This is where React mounts the entire application.

## Step 7: Set Up Tailwind CSS v4

Replace `src/index.css`:

```css
/* src/index.css */
@import "tailwindcss";

@theme {
  --font-sans: 'Inter', system-ui, sans-serif;
}

body {
  margin: 0;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* custom scrollbar for dark theme */
::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: #1a1a2e;
}

::-webkit-scrollbar-thumb {
  background: #4a4a6a;
  border-radius: 4px;
}
```

### Tailwind v4 vs v3 - What Changed?

In Tailwind v3, you needed:
- A `tailwind.config.js` file
- A `postcss.config.js` file
- CSS directives: `@tailwind base; @tailwind components; @tailwind utilities;`

In Tailwind v4, all of that is replaced with just:

```css
@import "tailwindcss";
```

Customization uses `@theme` blocks directly in CSS instead of a config file:

```css
@theme {
  --font-sans: 'Inter', system-ui, sans-serif;
}
```

This sets the default sans-serif font for all Tailwind's `font-sans` utilities.

## Step 8: Create the Entry Point

Replace `src/main.tsx`:

```typescript
// src/main.tsx
// purpose: react app entry point
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './index.css';
import { App } from './App';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

### How It Works

- `createRoot` is React 18+'s rendering API (replaces the old `ReactDOM.render`)
- `document.getElementById('root')!` - the `!` is a TypeScript non-null assertion. We know this element exists because we put it in `index.html`
- `StrictMode` enables extra development warnings. It double-invokes effects and renders to help you find bugs. It has zero impact in production
- We import `index.css` here so Tailwind is available everywhere

## Step 9: Create the Vite Type Declaration

Create `src/vite-env.d.ts`:

```typescript
// src/vite-env.d.ts
/// <reference types="vite/client" />
```

This single line tells TypeScript about Vite-specific features like `import.meta.env` and importing CSS/image files. Without it, TypeScript would show errors when you import `.css` files or use environment variables.

## Step 10: Create a Placeholder App

Replace `src/App.tsx`:

```typescript
// src/App.tsx
// purpose: main app shell (placeholder for now)

export const App = () => {
  return (
    <div className="min-h-screen bg-gray-950 text-white flex items-center justify-center">
      <div className="text-center">
        <h1 className="text-4xl font-bold mb-4">AI Component Builder</h1>
        <p className="text-gray-400">Project setup complete. Ready for Class 2.</p>
      </div>
    </div>
  );
};
```

Notice: we use `export const App` (named export), not `export default`. Named exports are easier to refactor and your editor can auto-import them.

## Step 11: Environment Variables

Create `.env.example`:

```
# firebase configuration
VITE_FIREBASE_API_KEY=your_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_project.firebasestorage.app
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=your_app_id
```

### How Vite Environment Variables Work

- Variables must start with `VITE_` to be exposed to the browser
- Access them with `import.meta.env.VITE_FIREBASE_API_KEY`
- Create a `.env` file (copy from `.env.example`) with your real values
- `.env` is gitignored so your keys never end up in version control

## Step 12: Set Up .gitignore

```
# .gitignore
# Logs
logs
*.log
npm-debug.log*

node_modules
dist
dist-ssr
*.local
*.tsbuildinfo

# Environment
.env
.env.local

# Editor directories and files
.vscode/*
!.vscode/extensions.json
.idea
.DS_Store
*.sw?
```

The critical lines are `.env` and `.env.local` - these prevent your API keys from being committed to git.

## Step 13: Run the Dev Server

```bash
npm run dev
```

Open `http://localhost:5173` in your browser. You should see a dark page with "AI Component Builder" centered on screen. Try editing the text in `App.tsx` and saving - notice how the browser updates instantly without a full page reload. That's Vite's HMR in action.

---

## Key Concepts Recap

**Vite** - A modern build tool that's fast because it uses native ES modules in development and only bundles for production.

**TypeScript Strict Mode** - Catches type errors at compile time. The `strict: true` flag enables all strict checks. This means you can't accidentally pass `null` where a string is expected, or forget to handle an error case.

**Tailwind CSS v4** - A utility-first CSS framework. Instead of writing CSS classes like `.card-header { padding: 16px; }`, you write `className="p-4"` directly in JSX. v4 simplified configuration by removing the need for config files.

**Environment Variables** - Values that change between environments (development, staging, production). They keep secrets like API keys out of your source code.

**Named Exports** - `export const App` instead of `export default App`. Named exports force you to import with the exact name, making refactoring safer and enabling better editor autocompletion.

---

## What's Next

In Class 2, we'll build the 3-panel layout (sidebar, preview area, gallery), define our TypeScript types, and create the interactive prompt input form - all with Tailwind dark theme styling.
