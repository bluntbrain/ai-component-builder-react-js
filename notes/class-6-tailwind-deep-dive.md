# Class 6: Tailwind CSS Deep Dive

## What You'll Learn

- How utility-first CSS works and why it's different from traditional CSS
- Customizing Tailwind v4 with `@theme`
- Responsive design with breakpoint prefixes
- Flexbox and Grid layouts in Tailwind
- Dark theme color patterns
- Animations, transitions, and interactive states
- The group hover pattern for card overlays
- Real patterns from our AI Component Builder project

## Why This Class Matters

You've been writing Tailwind classes throughout the AI Component Builder, but you might be copy-pasting them without fully understanding what each class does. This class gives you the mental model to build any layout from scratch.

---

## Utility-First CSS vs Traditional CSS

### Traditional CSS

In traditional CSS, you create custom class names and write styles in a separate file:

```css
/* styles.css */
.card {
  background-color: #1a1a2e;
  border-radius: 12px;
  padding: 16px;
  border: 1px solid #2e2e4a;
}

.card:hover {
  border-color: #4a4a6a;
}

.card-title {
  font-size: 14px;
  font-weight: 600;
  color: white;
}
```

```html
<div class="card">
  <h3 class="card-title">Hello</h3>
</div>
```

Problems:
- You need to invent class names for everything
- Styles live in a separate file, so you're constantly switching between files
- CSS grows endlessly because you're afraid to delete styles (what if something breaks?)
- Two developers might name the same thing differently

### Utility-First CSS (Tailwind)

With Tailwind, you compose small, single-purpose classes directly in your HTML/JSX:

```jsx
<div className="bg-gray-900 rounded-xl p-4 border border-gray-800 hover:border-gray-700">
  <h3 className="text-sm font-semibold text-white">Hello</h3>
</div>
```

Benefits:
- No naming. No switching files. What you see is what you get.
- Deleting a component deletes its styles automatically
- Every developer uses the same vocabulary (`p-4` is always 16px)
- The final CSS bundle only includes classes you actually use

---

## Tailwind v4 Customization with @theme

In Tailwind v4, customization happens directly in your CSS file using `@theme`:

```css
/* src/index.css */
@import "tailwindcss";

@theme {
  --font-sans: 'Inter', system-ui, sans-serif;
  --color-brand: #8b5cf6;
  --color-brand-dark: #7c3aed;
}
```

Now you can use these anywhere:

```jsx
<p className="font-sans">Uses Inter font</p>
<button className="bg-brand hover:bg-brand-dark">Custom color</button>
```

In Tailwind v3, you needed a `tailwind.config.js` file for this. v4 eliminated that file entirely.

---

## The Spacing and Sizing Scale

Tailwind uses a consistent 4px base unit:

| Class | Value | Pixels |
|-------|-------|--------|
| `p-1` | 0.25rem | 4px |
| `p-2` | 0.5rem | 8px |
| `p-3` | 0.75rem | 12px |
| `p-4` | 1rem | 16px |
| `p-6` | 1.5rem | 24px |
| `p-8` | 2rem | 32px |
| `p-12` | 3rem | 48px |

This works for all spacing utilities: `m-` (margin), `p-` (padding), `gap-` (flex/grid gap), `w-` (width), `h-` (height).

You can also use arbitrary values with square brackets:

```jsx
<div className="p-[13px]">Custom 13px padding</div>
<div className="w-[72%]">72% width</div>
```

---

## Responsive Design

Tailwind uses a mobile-first approach. Classes without a prefix apply to all screen sizes. Prefix classes apply at that breakpoint and above.

| Prefix | Min Width | Typical Device |
|--------|-----------|---------------|
| (none) | 0px | Mobile (default) |
| `sm:` | 640px | Large phone |
| `md:` | 768px | Tablet |
| `lg:` | 1024px | Laptop |
| `xl:` | 1280px | Desktop |
| `2xl:` | 1536px | Large monitor |

### Example: Responsive Grid

```jsx
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
  <div>Card 1</div>
  <div>Card 2</div>
  <div>Card 3</div>
</div>
```

This creates:
- **Mobile**: 1 column (cards stacked)
- **640px+**: 2 columns
- **1024px+**: 3 columns

### From Our Project: The Main Layout

```jsx
<div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
```

On mobile, the prompt input and gallery stack vertically. On laptops and above, they sit side by side.

---

## Flexbox with Tailwind

Flexbox is for 1-dimensional layouts (row or column).

### Horizontal layout (default)

```jsx
<div className="flex items-center gap-3">
  <img className="w-8 h-8 rounded-full" />
  <span>Username</span>
  <button className="ml-auto">Settings</button>  {/* pushes to right */}
</div>
```

| Class | What It Does |
|-------|-------------|
| `flex` | Display flex (horizontal by default) |
| `flex-col` | Stack vertically instead |
| `items-center` | Vertically center children |
| `justify-between` | Space children to edges |
| `justify-center` | Horizontally center children |
| `gap-3` | 12px gap between children |
| `flex-1` | Child takes all remaining space |
| `shrink-0` | Prevent child from shrinking |

### From Our Project: Sidebar Layout

```jsx
<aside className="w-72 bg-gray-900 border-r border-gray-800 flex flex-col h-screen sticky top-0">
  <div className="px-4 py-4 border-b border-gray-800">Logo</div>      {/* fixed height */}
  <div className="flex-1 overflow-y-auto px-4 py-4">Content</div>       {/* scrollable */}
  <div className="px-4 py-3 border-t border-gray-800">Footer</div>      {/* fixed height */}
</aside>
```

- `flex flex-col` stacks children vertically
- `flex-1` on the middle section makes it take all available height
- `overflow-y-auto` adds a scrollbar when content overflows
- `sticky top-0` keeps the sidebar fixed while the main content scrolls

---

## CSS Grid with Tailwind

Grid is for 2-dimensional layouts (rows AND columns).

```jsx
{/* 2 columns with gap */}
<div className="grid grid-cols-2 gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
  <div>Item 4</div>
</div>

{/* Item spanning 2 columns */}
<div className="grid grid-cols-3 gap-4">
  <div className="col-span-2">Wide item</div>
  <div>Normal item</div>
</div>
```

---

## Color System and Dark Theme

Tailwind's gray scale goes from 50 (lightest) to 950 (darkest):

```
bg-gray-50   -> #f9fafb (almost white)
bg-gray-100  -> #f3f4f6
bg-gray-200  -> #e5e7eb
bg-gray-300  -> #d1d5db
bg-gray-400  -> #9ca3af
bg-gray-500  -> #6b7280 (medium gray)
bg-gray-600  -> #4b5563
bg-gray-700  -> #374151
bg-gray-800  -> #1f2937
bg-gray-900  -> #111827 (very dark)
bg-gray-950  -> #030712 (almost black)
```

### Our Dark Theme Pattern

```
Background layers (darkest to lightest):
  bg-gray-950  -> main page background
  bg-gray-900  -> cards, sidebars
  bg-gray-800  -> input fields, buttons
  bg-gray-700  -> borders, dividers (border-gray-700)

Text hierarchy:
  text-white      -> headings, primary text
  text-gray-300   -> labels, secondary text
  text-gray-400   -> button text, metadata
  text-gray-500   -> placeholders, hints
  text-gray-600   -> timestamps, fine print

Accent colors:
  violet-500/600  -> primary action (buttons, focus rings)
  emerald-600     -> success action (save button)
  red-400         -> error messages
```

### Semi-transparent backgrounds

```jsx
<div className="bg-gray-950/50">  {/* 50% opacity black */}
<div className="bg-black/40">     {/* 40% opacity black, used for overlays */}
<div className="bg-red-500/10">   {/* 10% opacity red, subtle error background */}
```

---

## Interactive States

### Hover

```jsx
<button className="bg-violet-600 hover:bg-violet-700">
  Darkens on hover
</button>

<div className="border-gray-800 hover:border-gray-600">
  Border lightens on hover
</div>
```

### Focus

```jsx
<input className="focus:outline-none focus:ring-2 focus:ring-violet-500 focus:border-transparent" />
```

This removes the browser default outline and adds a violet glow ring.

### Disabled

```jsx
<button
  disabled={isLoading}
  className="bg-violet-600 disabled:opacity-50 disabled:cursor-not-allowed"
>
  {isLoading ? 'Loading...' : 'Submit'}
</button>
```

### Combining States

```jsx
<button className="bg-gray-800 text-gray-400 hover:text-white hover:border-gray-600 disabled:opacity-50 transition-colors">
```

The `transition-colors` class adds a smooth 150ms transition to color changes.

---

## The Group Hover Pattern

This is used in our gallery thumbnails. When you hover over a card, a copy button appears:

```jsx
<div className="group rounded-lg border border-gray-800 overflow-hidden hover:border-gray-600">
  {/* thumbnail */}
  <div className="relative">
    <img src="..." />
    {/* overlay appears on hover */}
    <div className="absolute inset-0 bg-black/0 group-hover:bg-black/40 flex items-center justify-center">
      <button className="opacity-0 group-hover:opacity-100">
        Copy
      </button>
    </div>
  </div>
</div>
```

How it works:
1. `group` on the parent marks it as a hover target
2. `group-hover:bg-black/40` on the overlay div - shows dark background when PARENT is hovered
3. `group-hover:opacity-100` on the button - fades in when PARENT is hovered

Without `group`, you'd need JavaScript to detect the parent hover and toggle child styles.

---

## Animations

### Built-in Animations

```jsx
{/* loading spinner */}
<div className="w-8 h-8 border-2 border-violet-500 border-t-transparent rounded-full animate-spin" />

{/* skeleton loading */}
<div className="h-4 bg-gray-800 rounded animate-pulse" />

{/* bouncing dots */}
<span className="animate-bounce" />
```

### Transitions

```jsx
{/* smooth color transition (150ms default) */}
<button className="transition-colors hover:bg-violet-700">

{/* smooth all transitions */}
<div className="transition-all hover:scale-105">

{/* smooth opacity */}
<div className="transition-opacity opacity-0 group-hover:opacity-100">
```

---

## Common Patterns Cheat Sheet

### Center anything

```jsx
{/* center horizontally and vertically */}
<div className="flex items-center justify-center h-screen">
  <p>Centered</p>
</div>
```

### Truncate text

```jsx
<p className="truncate">This very long text will be cut off with an ellipsis...</p>
```

### Sticky header

```jsx
<header className="sticky top-0 z-10 bg-gray-950/80 backdrop-blur-sm">
```

### Gradient button

```jsx
<button className="bg-linear-to-r from-violet-600 to-indigo-600 hover:from-violet-700 hover:to-indigo-700">
```

### Aspect ratio

```jsx
<div className="aspect-video">  {/* 16:9 */}
<div className="aspect-square"> {/* 1:1 */}
<div className="aspect-[3/4]">  {/* custom 3:4 ratio */}
```

### Custom values with brackets

```jsx
<div className="w-[calc(100%-2rem)]">
<div className="grid-cols-[200px_1fr_200px]">
<div className="text-[#ff6b6b]">
```

---

## Quick Reference: Classes Used in Our Project

| Class | Used In | Purpose |
|-------|---------|---------|
| `w-72` | Sidebar | Fixed width 288px |
| `w-48` | Gallery sidebar | Fixed width 192px |
| `h-screen` | Sidebars | Full viewport height |
| `sticky top-0` | Sidebars | Fixed position on scroll |
| `flex-1` | Preview panel | Takes remaining width |
| `min-h-0` | Preview panel | Allows flex child to shrink below content size |
| `overflow-y-auto` | Scrollable areas | Scrollbar when needed |
| `bg-gray-950` | Page background | Darkest background |
| `bg-gray-900` | Cards, sidebars | Dark card background |
| `border-gray-800` | Borders | Subtle border |
| `rounded-xl` | Cards | 12px border radius |
| `rounded-lg` | Buttons, inputs | 8px border radius |
| `animate-spin` | Spinner | Rotation animation |
| `animate-pulse` | Skeletons | Fade animation |
| `pointer-events-none` | Thumbnails | Disable clicking |
| `select-none` | Line numbers | Disable text selection |

---

## Key Concepts Recap

**Utility-first** means composing styles from small, single-purpose classes instead of writing custom CSS. It feels wrong at first ("these classNames are so long!") but eliminates CSS bloat, naming debates, and the fear of deleting styles.

**Mobile-first** means writing base styles for mobile, then adding breakpoint prefixes (`md:`, `lg:`) for larger screens. Think "stack by default, spread out when there's room."

**The spacing scale** is consistent: `1` = 4px, `2` = 8px, `4` = 16px, `8` = 32px. Once you internalize this, you stop thinking in pixels.

**Transitions** should be on the base element, not on the hover state. Write `transition-colors` once, and all color changes (hover, focus, disabled) will animate smoothly.

---

## What's Next

In Class 7, we'll learn Git and GitHub - the version control system every professional developer uses. You'll learn how to track changes, create branches, push code, and collaborate through pull requests.
