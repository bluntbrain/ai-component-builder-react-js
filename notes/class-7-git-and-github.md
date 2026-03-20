# Class 7: Git & GitHub Essentials

## What You'll Learn

- What version control is and why every developer needs it
- Core Git commands: init, add, commit, status, log, diff
- Writing meaningful commit messages
- Branching and merging
- Pushing code to GitHub
- Creating pull requests
- Common mistakes and how to fix them
- A practical Git workflow for your own projects

## Why This Class Matters

Every job posting for a developer mentions Git. Every team uses it. If you can't use Git confidently, you'll struggle in any professional setting. This class teaches you the 20% of Git that covers 95% of daily usage.

---

## What is Git?

Git is a version control system. It tracks every change you make to your code over time, like a save history for your entire project.

Without Git:
- You make a change that breaks something. You can't go back.
- You try to collaborate with a teammate. You email zip files back and forth.
- You have no record of what changed, when, or why.

With Git:
- Every change is recorded as a "commit" with a message explaining why
- You can go back to any previous state of your project
- Multiple people can work on the same codebase without conflicts
- You have a complete history of your project

**Git** runs on your computer (local). **GitHub** is a website that hosts your Git repositories online so others can see and collaborate on your code.

---

## Setting Up Git

### First-time setup (run once)

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

This sets your identity for all commits.

---

## Core Git Workflow

The daily workflow is: **Edit -> Stage -> Commit -> Push**

### Step 1: Initialize a Repository

```bash
mkdir my-project
cd my-project
git init
```

This creates a hidden `.git/` folder that tracks everything. You only do this once per project.

### Step 2: Check Status

```bash
git status
```

This shows you:
- **Untracked files**: new files Git doesn't know about (red)
- **Modified files**: files you've changed (red)
- **Staged files**: files ready to be committed (green)

Run `git status` constantly. It's your compass.

### Step 3: Stage Files

```bash
# stage specific files
git add src/App.tsx src/types.ts

# stage all changed files
git add .
```

Staging means "I want to include these changes in my next commit." Think of it as putting items in a box before shipping.

### Step 4: Commit

```bash
git commit -m "add prompt input component with example chips"
```

A commit is a snapshot of your project at a point in time. The `-m` flag adds a message describing what changed.

### Step 5: View History

```bash
git log --oneline
```

Output:
```
a1b2c3d add prompt input component with example chips
e4f5g6h set up vite project with tailwind v4
9h8i7j6 initial commit
```

Each line is a commit with a unique ID and your message.

---

## Writing Good Commit Messages

Bad commit messages:
```
"fixed stuff"
"update"
"asdfgh"
"final version"
"final version 2"
"final FINAL version"
```

Good commit messages:
```
"add login form with email validation"
"fix sidebar overflow on mobile screens"
"integrate OpenAI API for component generation"
"update gallery to show skeleton loading state"
"remove unused appwrite dependency"
```

### Rules

1. Start with a verb: add, fix, update, remove, refactor
2. Be specific: what did you change and why?
3. Keep it under 72 characters
4. Don't explain "how" (the code diff shows that) - explain "what" and "why"

---

## .gitignore

The `.gitignore` file tells Git to ignore certain files and folders:

```
# dependencies - never commit these (thousands of files)
node_modules

# build output - can be regenerated
dist

# secrets - NEVER commit API keys
.env
.env.local

# OS files
.DS_Store
```

**Critical rule**: Never commit API keys, passwords, or secrets. If you accidentally commit a secret, consider it compromised - change the key immediately.

---

## Branching

Branches let you work on features without affecting the main code.

```
main --------o--------o--------o
              \                 /
feature        o----o----o----o
```

### Create and Switch to a Branch

```bash
# create and switch to a new branch
git checkout -b feature/add-gallery

# see all branches
git branch

# switch back to main
git checkout main
```

### Merge a Branch

When your feature is done:

```bash
# switch to main
git checkout main

# merge the feature branch into main
git merge feature/add-gallery

# delete the feature branch (optional, it's merged)
git branch -d feature/add-gallery
```

### When to Branch

- Adding a new feature
- Fixing a bug
- Experimenting with something risky

Rule of thumb: if the change takes more than an hour, make a branch.

---

## GitHub

### Create a Repository on GitHub

1. Go to github.com and click "New repository"
2. Name it (e.g., `ai-component-builder`)
3. Do NOT initialize with README (you already have code)
4. Click "Create repository"

### Push Local Code to GitHub

```bash
# add GitHub as a remote (run once)
git remote add origin https://github.com/YOUR_USERNAME/ai-component-builder.git

# push your code
git push -u origin main
```

The `-u` flag sets up tracking so future pushes only need `git push`.

### Push Changes After Commits

```bash
git add .
git commit -m "add gallery sidebar with thumbnails"
git push
```

### Clone Someone Else's Repository

```bash
git clone https://github.com/someone/their-project.git
cd their-project
npm install
```

This downloads the full repository with all its history.

---

## Pull Requests (PRs)

A pull request is how you propose changes on GitHub. Instead of pushing directly to `main`, you:

1. Create a branch
2. Make your changes and commit
3. Push the branch to GitHub
4. Open a pull request on GitHub
5. Someone reviews your code
6. Merge the PR

### Creating a PR

```bash
# create a branch and make changes
git checkout -b feature/dark-mode
# ... make changes ...
git add .
git commit -m "add dark mode toggle to header"
git push -u origin feature/dark-mode
```

Then go to GitHub. You'll see a "Compare & pull request" button. Click it, write a description, and submit.

### Why PRs Matter

- Code gets reviewed before merging (catches bugs)
- There's a record of what changed and why
- Other developers can comment and suggest improvements
- This is how every professional team works

---

## Common Mistakes and Fixes

### "I committed something I shouldn't have"

```bash
# undo the last commit but keep the changes
git reset --soft HEAD~1
```

### "I want to see what I changed"

```bash
# see unstaged changes
git diff

# see staged changes
git diff --staged
```

### "I made changes but want to throw them away"

```bash
# discard changes in a specific file
git checkout -- src/App.tsx

# discard ALL uncommitted changes (careful!)
git stash
```

`git stash` saves your changes and gives you a clean slate. You can get them back with `git stash pop`.

### "I'm on the wrong branch"

```bash
# save current changes
git stash

# switch branch
git checkout main

# apply saved changes here
git stash pop
```

### "I need to see what happened"

```bash
# see all commits
git log --oneline

# see who changed each line of a file
git blame src/App.tsx

# see what changed in a specific commit
git show a1b2c3d
```

---

## Git Workflow for Your Own Project

Here's the workflow you should follow for your project over the next 15 days:

### Day 1: Setup

```bash
mkdir my-project
cd my-project
git init
npm create vite@latest . -- --template react-ts
npm install
git add .
git commit -m "initial project setup with vite and react"
```

Create a GitHub repo and push:

```bash
git remote add origin https://github.com/YOU/my-project.git
git push -u origin main
```

### Every Day: Commit Often

```bash
# after completing each feature or fixing something
git add .
git commit -m "add user authentication with firebase"
git push
```

Commit at least once per work session. Small, frequent commits are better than one massive commit.

### Good Commit Frequency

```
Day 1:  "initial project setup"
Day 1:  "add basic layout with navigation"
Day 2:  "integrate weather API and display data"
Day 2:  "add loading and error states"
Day 3:  "style dashboard with tailwind dark theme"
Day 4:  "add search functionality"
Day 4:  "fix search not clearing on new query"
...
```

---

## Command Cheat Sheet

| Command | What It Does |
|---------|-------------|
| `git init` | Start tracking a project |
| `git status` | See what's changed |
| `git add .` | Stage all changes |
| `git add file.tsx` | Stage a specific file |
| `git commit -m "msg"` | Save a snapshot with a message |
| `git log --oneline` | See commit history |
| `git diff` | See unstaged changes |
| `git branch` | List branches |
| `git checkout -b name` | Create and switch to a branch |
| `git checkout main` | Switch to main branch |
| `git merge branch-name` | Merge a branch into current |
| `git remote add origin URL` | Connect to GitHub |
| `git push` | Upload commits to GitHub |
| `git pull` | Download changes from GitHub |
| `git clone URL` | Download a repository |
| `git stash` | Temporarily save uncommitted changes |
| `git stash pop` | Restore stashed changes |

---

## Key Concepts Recap

**Commits** are snapshots. Each one records what changed and why. They're permanent and can be revisited at any time.

**Branches** are parallel timelines. Create one for each feature. Merge it back when done. This keeps `main` stable.

**GitHub** is the online home for your code. Push your code there so it's backed up, shareable, and ready for deployment.

**Pull Requests** are how teams review code. Even working solo, they're useful for organizing your work and keeping a clean history.

**Commit often**. A commit should represent one logical change. "Add login page" is one commit. "Add login page, fix navbar, update footer, add 3 new pages" should be 4 separate commits.

---

## What's Next

In Class 8, we'll switch from OpenAI (paid) to Gemini (free) for our AI Component Builder. You'll learn how to swap API providers with minimal code changes.
