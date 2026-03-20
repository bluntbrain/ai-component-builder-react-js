# Class 10: Choose Your Own Project + Portfolio Website

## What You'll Learn

- How to choose a project that's the right scope for 15 days
- How to submit your project idea for approval
- Project ideas with difficulty levels
- Why every developer needs a portfolio website
- How to buy a domain and connect it to Vercel
- How to record short demo videos of your projects

## Important Dates

| Date | What |
|------|------|
| **Thursday, March 26** | Submit your project idea (WhatsApp group or Notion doc) |
| **March 26 - April 10** | Build your project (15 days) |
| **April 10** | Final deadline: project + portfolio website live on your domain |

---

# Part A: Choose Your Own Project

## What You Need to Submit

By **Thursday, March 26**, post in the WhatsApp group or create a Notion doc with:

1. **Project name** - what you're calling it
2. **One-line description** - what it does in one sentence
3. **Detailed description** - 3-5 sentences explaining the full idea
4. **Tech stack** - React + TypeScript + Tailwind + which APIs/services
5. **Key features** - bullet list of 4-6 features you'll build
6. **Why this project** - what interests you about it, what you'll learn

Ishan will review each submission and either approve it or suggest adjustments to make sure it's achievable in 15 days.

## Requirements

Your project must include:

- React + TypeScript (what we've been learning)
- Tailwind CSS for styling
- At least one external API integration (AI, weather, maps, finance, etc.)
- Multiple components with props
- State management (useState, useEffect, useCallback)
- Loading and error states
- Deployed to Vercel

## What Makes a Good Project

**Good scope**: Can be built in 15 days working a few hours daily. Has a clear "done" state - you know when it's finished.

**Solves a problem**: Even a small one. "I want to quickly translate text" or "I want to compare crypto prices" is enough.

**Shows React skills**: Uses multiple hooks, has several components, handles async data, shows different UI states.

**Has an API**: Demonstrates you can work with external data. AI APIs, REST APIs, Firebase - all count.

**Looks polished**: A clean, dark-themed UI with loading states and error handling impresses more than a feature-heavy app that looks broken.

## Project Ideas

Here are ideas organized by difficulty. Pick one or use them as inspiration for your own idea.

### Beginner-Friendly

**1. AI Resume Builder**
- User fills in their details (name, experience, skills, education)
- AI generates a professional resume/summary
- Preview in a formatted template
- Download or copy the result
- **API**: Gemini for text generation
- **What you'll learn**: Form handling, structured data, PDF generation

**2. Weather Dashboard**
- Search for any city
- Shows current weather + 5-day forecast
- Beautiful weather-themed UI (sunny/rainy backgrounds)
- Save favorite cities to localStorage
- **API**: OpenWeatherMap (free tier)
- **What you'll learn**: REST APIs, data visualization, localStorage

**3. Quote Generator + Collection**
- Generate motivational/funny quotes by category
- Save favorites to Firebase
- Share quotes as images
- Daily quote notification concept
- **API**: Gemini or a quotes API
- **What you'll learn**: Firebase, category filtering, sharing

### Intermediate

**4. AI Flashcard Study App**
- User enters a topic, AI generates flashcards
- Flip animation to reveal answers
- Track which cards you've mastered
- Spaced repetition logic
- **API**: Gemini for flashcard generation
- **What you'll learn**: Animations, local state tracking, study algorithms

**5. Expense Tracker with AI Insights**
- Add daily expenses (amount, category, date)
- Charts showing spending by category
- AI analyzes your spending and gives advice
- Monthly summaries
- **API**: Gemini for insights, Firebase for storage
- **What you'll learn**: Charts, data aggregation, CRUD operations

**6. Code Snippet Manager**
- Save code snippets with titles and tags
- Syntax highlighting for multiple languages
- Search and filter by language/tag
- AI explains what a snippet does
- **API**: Gemini for code explanation, Firebase for storage
- **What you'll learn**: Search, filtering, syntax highlighting

**7. Movie/Book Recommendation App**
- Search for movies or books
- AI suggests similar titles based on what you like
- Save a "watchlist" or "reading list"
- Rate items you've seen/read
- **API**: TMDB or Google Books + Gemini
- **What you'll learn**: Search APIs, recommendations, lists

### Advanced

**8. Real-time Chat with AI Assistant**
- Chat interface like WhatsApp/Telegram
- AI assistant in the chat that can answer questions
- Message history saved to Firebase
- Multiple chat rooms/topics
- **API**: Gemini for AI responses, Firebase for real-time data
- **What you'll learn**: Real-time updates, chat UI, message queuing

**9. AI-Powered Blog Editor**
- Write blog posts with a rich text area
- AI helps with grammar, tone, and suggestions
- Preview the post as it would look published
- Save drafts to Firebase, publish publicly
- **API**: Gemini for writing assistance, Firebase for storage
- **What you'll learn**: Rich text editing, drafts, publishing flow

**10. Portfolio Website Builder**
- Users fill in their info and projects
- AI generates a complete portfolio page
- Multiple template options
- Live preview (similar to our AI Component Builder)
- Export as HTML
- **API**: Gemini for content generation
- **What you'll learn**: Templates, HTML generation, export

**11. Crypto/Stock Price Tracker**
- Search and track crypto or stock prices
- Real-time price updates
- Price charts with historical data
- Set price alerts (local notifications)
- **API**: CoinGecko (crypto) or Alpha Vantage (stocks)
- **What you'll learn**: WebSockets/polling, charts, real-time data

**12. AI Trip Planner**
- Enter a destination and number of days
- AI generates a day-by-day itinerary
- Show places on a map
- Save and edit itineraries
- **API**: Gemini for planning, Google Maps embed, Firebase for saving
- **What you'll learn**: Maps integration, structured AI output, CRUD

---

## How to Start Your Project

### Day 1: Setup and Layout

```bash
mkdir my-project-name
cd my-project-name
npm create vite@latest . -- --template react-ts
npm install
npm install -D tailwindcss @tailwindcss/vite
# install your specific packages
git init
git add .
git commit -m "initial project setup"
```

### Day 2-3: Core UI

Build the layout and all component shells with placeholder data. Get the visual design right before adding API logic.

### Day 4-7: API Integration

Connect your APIs. Handle loading, error, and success states. Get data flowing.

### Day 8-10: Features

Add search, filtering, saving, or whatever features your project needs.

### Day 11-13: Polish

Fix bugs, improve UX, add animations, handle edge cases.

### Day 14-15: Deploy and Record

Deploy to Vercel. Record your demo video. Add to portfolio.

---

# Part B: Portfolio Website

## Why You Need a Portfolio

When you apply for jobs or freelance work, people look at:

1. Your portfolio website (do you have real projects?)
2. Your GitHub (is your code clean?)
3. Your LinkedIn (do you have the right experience?)

A portfolio website with a custom domain shows you're serious. `yourname.dev` looks far more professional than a Google Docs resume.

## Step 1: Buy a Domain

Recommended domain registrars (cheapest options):

| Registrar | Price/Year | Notes |
|-----------|-----------|-------|
| Namecheap | $6-12 | Good UI, free privacy |
| Cloudflare | $8-10 | No markup, best DNS |
| GoDaddy | $10-15 | Popular but has upsells |

Good domain patterns:
- `yourname.dev` (best for developers)
- `yourname.com`
- `yourname.in` (for India, very cheap)
- `yourname.tech`

Tips:
- Keep it short and professional
- Use your real name
- `.dev` domains require HTTPS (Vercel handles this automatically)

## Step 2: Build Your Portfolio

Your portfolio doesn't need to be complex. A single-page site with these sections is enough:

### Required Sections

1. **Hero/About** - Your name, one-line description ("React Developer"), a short bio
2. **Projects** - Cards for each project with:
   - Project name
   - Short description
   - Screenshot or short video
   - Links: live demo + GitHub repo
3. **Skills** - Tech stack icons/badges (React, TypeScript, Tailwind, Firebase, etc.)
4. **Contact** - Email, LinkedIn, GitHub links

### Your Two Projects

| Project | What It Shows |
|---------|--------------|
| AI Component Builder | AI integration, iframe sandboxing, Firebase, structured outputs |
| Your chosen project | Your creativity, independent problem-solving, new APIs |

## Step 3: Record Demo Videos

For each project, record a 30-60 second video showing:

1. The app loading
2. The main feature in action (e.g., generating a component)
3. Any cool interactions (copy button, gallery, etc.)

### How to Record

- **Mac**: Press Cmd+Shift+5 for screen recording
- **Windows**: Press Win+G for Game Bar recording
- **Any OS**: Use [Loom](https://www.loom.com) (free, easy sharing)
- **For GIFs**: Use [Kap](https://getkap.co) (Mac) or [ScreenToGif](https://www.screentogif.com) (Windows)

Keep videos short. Recruiters spend 10-30 seconds per project. Show the best parts fast.

## Step 4: Deploy to Vercel

If you haven't already:

```bash
npm install -g vercel
vercel
```

Or push to GitHub and import the repo on [vercel.com](https://vercel.com).

## Step 5: Connect Your Domain

1. In Vercel dashboard, go to your project **Settings > Domains**
2. Add your domain (e.g., `yourname.dev`)
3. Vercel will show you DNS records to add
4. Go to your domain registrar's DNS settings
5. Add the records Vercel shows you:
   - Usually an `A` record pointing to `76.76.21.21`
   - And a `CNAME` record for `www` pointing to `cname.vercel-dns.com`
6. Wait 10-30 minutes for DNS propagation
7. Vercel automatically provisions an SSL certificate (HTTPS)

After setup, your portfolio will be live at `https://yourname.dev`.

---

## Final Checklist

Before the April 10 deadline, you should have:

- [ ] AI Component Builder project deployed and working
- [ ] Your chosen project idea approved by Ishan
- [ ] Your chosen project built and deployed to Vercel
- [ ] A domain purchased and connected
- [ ] Portfolio website live on your custom domain
- [ ] Short demo video for each project embedded in portfolio
- [ ] Both projects linked from your portfolio with live demo + GitHub links
- [ ] Clean GitHub repos with meaningful commit history
- [ ] Both projects working without errors when someone visits the live URLs

## How You'll Be Evaluated

| Criteria | Weight |
|----------|--------|
| Project works and is deployed | 30% |
| Code quality (TypeScript, components, error handling) | 25% |
| UI/UX polish (looks professional, loading states, responsive) | 20% |
| Portfolio website with custom domain | 15% |
| Git history (regular commits, good messages) | 10% |

---

## Final Advice

1. **Start simple, then add features**. A working app with 3 features is better than a broken app with 10.

2. **Commit every day**. Your git history shows your work ethic.

3. **Ask for help early**. If you're stuck for more than 30 minutes, ask in the WhatsApp group.

4. **Focus on polish over features**. Loading spinners, error messages, smooth transitions - these small details separate amateur projects from professional ones.

5. **Ship it**. A deployed project that's 80% perfect is worth infinitely more than a local project that's 100% perfect but nobody can see.

Good luck. Build something you're proud of.
