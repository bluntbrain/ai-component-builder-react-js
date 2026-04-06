# Class 15: Git Mastery -- Live Exercises & Advanced Commands

## What You'll Do Today

This is a **hands-on class**. You will:

1. Watch the instructor create a repo live on GitHub
2. Fork the repo to your own account
3. Clone it, create a branch, add your name, commit, push
4. Open a Pull Request back to the original repo
5. Learn advanced Git commands (cherry-pick, rebase, stash, reset, etc.)

By the end, you'll have a real PR on a real repo -- and know the commands that separate a junior from a senior developer.

---

# Part A: The Live Exercise

---

## Exercise Setup (Instructor Does This)

**Step 1:** Instructor creates a new public repo on GitHub called `class-roster`

**Step 2:** Instructor adds a `README.md` and a `students/` folder with a template file:

```bash
mkdir class-roster && cd class-roster
git init
```

**Step 3:** Instructor creates the initial files:

`README.md`:
```markdown
# Class Roster

This repo is a live exercise. Each student will fork this repo,
add their name, and create a Pull Request.

## How to Participate

1. Fork this repo
2. Clone your fork
3. Create a branch
4. Add your file in the students/ folder
5. Commit, push, and open a PR
```

`students/_template.md`:
```markdown
# Your Name

- **GitHub:** @yourusername
- **One thing you learned this week:** ...
- **Favorite programming language:** ...
```

```bash
git add .
git commit -m "initial commit: add readme and template"
git branch -M main
git remote add origin https://github.com/INSTRUCTOR/class-roster.git
git push -u origin main
```

The repo is now live. Students can see it at `github.com/INSTRUCTOR/class-roster`.

---

## Exercise Steps (Students Follow Along)

### Step 1: Fork the Repo

```
  WHAT IS A FORK?

  Instructor's repo                      Your fork (your copy)
  github.com/INSTRUCTOR/class-roster     github.com/YOU/class-roster

  +-------------------+                  +-------------------+
  | README.md         |   --- fork --->  | README.md         |
  | students/         |                  | students/         |
  |   _template.md    |                  |   _template.md    |
  +-------------------+                  +-------------------+

  A fork is a FULL COPY of the repo under YOUR GitHub account.
  You have full write access to your fork.
  Changes in your fork don't affect the original.
```

1. Go to `github.com/INSTRUCTOR/class-roster`
2. Click the **Fork** button (top right)
3. Select your account
4. You now have `github.com/YOUR_USERNAME/class-roster`

### Step 2: Clone Your Fork

```bash
# clone YOUR fork (not the instructor's repo)
git clone https://github.com/YOUR_USERNAME/class-roster.git
cd class-roster
```

```
  CLONE vs FORK:

  Fork  = copy a repo on GitHub (server to server)
  Clone = download a repo to your computer (server to local)

  GitHub (cloud)                    Your laptop (local)
  +-------------------+            +-------------------+
  | YOUR_USERNAME/    |  -- clone ->| class-roster/     |
  | class-roster      |            | (full copy with   |
  +-------------------+            |  all git history) |
                                   +-------------------+
```

### Step 3: Create a Branch

```bash
# create and switch to a new branch
git checkout -b add-YOUR_NAME

# example:
git checkout -b add-ishan
```

```
  BRANCHING:

  main:       [commit 1] --- [commit 2] --- [commit 3]
                                               \
  add-ishan:                                    +--- [your changes here]

  Your branch is a separate line of work.
  main stays untouched while you work.
```

### Step 4: Add Your File

Create a new file `students/your-name.md`:

```bash
# copy the template
cp students/_template.md students/ishan.md
```

Edit `students/ishan.md` with your actual info:

```markdown
# Ishan

- **GitHub:** @ishandev
- **One thing I learned this week:** How the event loop works in JavaScript
- **Favorite programming language:** TypeScript
```

### Step 5: Stage, Commit, Push

```bash
# check what changed
git status

# stage your file
git add students/ishan.md

# commit with a message
git commit -m "add ishan to class roster"

# push your branch to YOUR fork on GitHub
git push origin add-ishan
```

```
  WHAT JUST HAPPENED:

  Working        Staging          Local             Remote
  Directory      Area             Repository        (GitHub)

  [edit file] -> git add -> [staged] -> git commit -> [committed] -> git push -> [on GitHub]

  git add    = "I want to include this change in my next commit"
  git commit = "Save this snapshot with a message"
  git push   = "Upload my commits to GitHub"
```

### Step 6: Create a Pull Request

```
  PULL REQUEST FLOW:

  Your fork                             Instructor's repo
  (github.com/YOU/class-roster)         (github.com/INSTRUCTOR/class-roster)

  Branch: add-ishan                     Branch: main
  +-------------------+                 +-------------------+
  | students/ishan.md |  --- PR --->    | (review & merge)  |
  +-------------------+                 +-------------------+

  A PR says: "Hey, I made changes in my fork.
  Please review and merge them into the original repo."
```

1. Go to your fork on GitHub
2. You'll see a banner: **"add-ishan had recent pushes. Compare & pull request"**
3. Click **Compare & pull request**
4. Fill in:
   - **Title:** "Add Ishan to class roster"
   - **Description:** "Adding my info to the students directory"
5. Click **Create pull request**
6. Wait for the instructor to review and merge

### Step 7: Instructor Reviews and Merges

The instructor will:
1. Open each PR
2. Review the changes (click "Files changed" tab)
3. Leave a comment or approve
4. Click **Merge pull request**
5. Everyone's names are now in the main repo

---

# Part B: Essential Git Commands Reference

---

## The Basics (daily use)

```bash
# check current status (what's changed, what's staged)
git status

# see what you changed (unstaged changes)
git diff

# see what's staged (ready to commit)
git diff --staged

# stage specific files
git add file1.txt file2.txt

# stage all changes (use cautiously)
git add .

# commit with message
git commit -m "fix login button alignment"

# see commit history
git log --oneline

# push to remote
git push origin branch-name

# pull latest changes from remote
git pull origin main
```

---

## Branching

```bash
# list all branches (* = current)
git branch

# create and switch to new branch
git checkout -b feature/login-page

# switch to existing branch
git checkout main

# delete a branch (after merging)
git branch -d feature/login-page

# delete a branch (force, even if not merged)
git branch -D feature/login-page
```

```
  BRANCHING STRATEGY:

  main ----[v1.0]----------[v1.1]----------[v1.2]----
               \             /   \             /
  feature/      +--[work]--+     +--[work]--+
  login           (branch)         (branch)

  Rule: never commit directly to main.
  Always create a branch, do your work, then merge via PR.
```

---

## Merging

```bash
# merge a branch into current branch
git checkout main
git merge feature/login-page
```

```
  MERGE TYPES:

  FAST-FORWARD (no divergence):
  main:    A --- B
                  \
  feature:         C --- D

  After merge:
  main:    A --- B --- C --- D   (linear, clean)


  THREE-WAY MERGE (diverged):
  main:    A --- B --- E
                  \
  feature:         C --- D

  After merge:
  main:    A --- B --- E --- M   (M = merge commit)
                  \         /
  feature:         C --- D
```

---

## Merge Conflicts

When two people change the **same line** in the same file, Git can't auto-merge. You must resolve it manually.

```
  THE CONFLICT MARKERS:

  <<<<<<< HEAD
  const color = 'blue';        <-- your change (current branch)
  =======
  const color = 'red';         <-- their change (incoming branch)
  >>>>>>> feature/new-theme

  YOU DECIDE: keep one, keep both, or write something new.
  Then remove the markers, stage, and commit.
```

```bash
# after resolving conflicts in the file:
git add resolved-file.txt
git commit -m "resolve merge conflict in theme colors"
```

---

## Stash -- Save Work Without Committing

You're mid-feature but need to switch branches urgently. Stash saves your uncommitted changes temporarily.

```bash
# save current changes to stash
git stash

# your working directory is now clean -- switch branches freely
git checkout main
# do urgent work...
git checkout feature/login

# bring back your stashed changes
git stash pop

# list all stashes
git stash list

# stash with a name (useful when you have multiple)
git stash save "half-done login form"
```

```
  STASH ANALOGY:

  You're cooking dinner (feature branch).
  Doorbell rings (urgent bug on main).

  git stash = put everything in the fridge (save for later)
  git checkout main = go answer the door
  (fix the bug, commit, push)
  git checkout feature = back to the kitchen
  git stash pop = take everything out of the fridge, continue cooking
```

---

## Cherry-Pick -- Copy a Specific Commit

Take ONE commit from another branch and apply it to your current branch. Doesn't merge the whole branch.

```bash
# find the commit hash you want
git log --oneline feature/payments
# a1b2c3d fix currency formatting bug
# d4e5f6g add payment gateway
# h7i8j9k update checkout page

# cherry-pick just the bug fix
git checkout main
git cherry-pick a1b2c3d
```

```
  CHERRY-PICK:

  feature:  A --- B --- C --- D
                  ^
                  |
            "I only want THIS commit"

  main:     X --- Y --- B'
                        ^
                   cherry-picked!
                   (B' is a copy of B)

  USE CASE: A critical bug fix is on a feature branch
  that isn't ready to merge. Cherry-pick just the fix.
```

---

## Rebase -- Rewrite History for a Clean Timeline

Rebase moves your branch's commits ON TOP of another branch, creating a linear history.

```bash
# you're on feature/login, main has moved ahead
git checkout feature/login
git rebase main
```

```
  BEFORE REBASE:

  main:      A --- B --- C
                  \
  feature:         D --- E    (branched from B, but main has C now)


  AFTER REBASE:

  main:      A --- B --- C
                          \
  feature:                 D' --- E'   (replayed on top of C)

  D' and E' are NEW commits (same changes, different hashes).
  History is now LINEAR -- as if you started from C.


  MERGE vs REBASE:

  Merge:   A --- B --- C --- M     (merge commit, shows branch existed)
                  \         /
                   D --- E

  Rebase:  A --- B --- C --- D' --- E'   (linear, no merge commit)

  Rebase = cleaner history, but NEVER rebase commits already pushed.
```

**Golden rule:** Never rebase commits that others have pulled. Only rebase your LOCAL, unpushed work.

---

## Reset -- Undo Commits

```bash
# soft reset: undo commit but keep changes staged
git reset --soft HEAD~1

# mixed reset (default): undo commit, unstage changes, keep in working dir
git reset HEAD~1

# hard reset: undo commit AND delete all changes (DESTRUCTIVE)
git reset --hard HEAD~1
```

```
  RESET TYPES:

  Before: [commit 1] --- [commit 2] --- [commit 3 (HEAD)]

                        --soft            --mixed           --hard
                        (gentle)          (default)         (nuclear)

  Commit history:       removes commit 3  removes commit 3  removes commit 3
  Staging area:         changes STAGED    changes UNSTAGED  changes DELETED
  Working directory:    changes KEPT      changes KEPT      changes DELETED

  Think of it as:
  --soft:  "I want to redo my commit message or combine commits"
  --mixed: "I want to restage my changes differently"
  --hard:  "I want to completely throw away these changes"
```

---

## Revert -- Undo a Commit Safely (for pushed code)

Unlike `reset`, `revert` creates a NEW commit that undoes a previous commit. Safe for shared branches.

```bash
# revert a specific commit (creates a new "undo" commit)
git revert a1b2c3d
```

```
  RESET vs REVERT:

  Reset (rewrites history):     Revert (adds new commit):

  A --- B --- C                 A --- B --- C --- C'
        (C is gone!)                        (C' undoes C)

  Reset: dangerous on shared branches (others still have C)
  Revert: safe on shared branches (history is preserved)

  Rule: use REVERT for pushed commits, RESET for local only.
```

---

## Reflog -- Your Safety Net

Git records EVERY action you take. Even if you accidentally `reset --hard`, reflog can save you.

```bash
# see everything you've done
git reflog

# output:
# a1b2c3d HEAD@{0}: reset: moving to HEAD~1
# f4e5d6c HEAD@{1}: commit: add payment form
# b7c8d9e HEAD@{2}: checkout: moving from main to feature

# recover the "lost" commit
git checkout f4e5d6c
# or
git reset --hard f4e5d6c
```

```
  REFLOG = your "undo history"

  Even after git reset --hard, the commit still EXISTS
  in Git's internal log for ~30 days.

  reflog is the "Ctrl+Z" of Git.
  If you think you lost work, check reflog FIRST.
```

---

## Interactive Rebase -- Clean Up Before a PR

Squash messy commits into clean ones before opening a PR.

```bash
# squash last 3 commits into one
git rebase -i HEAD~3
```

Git opens an editor:

```
pick a1b2c3d add login form
pick d4e5f6g fix typo in login form
pick h7i8j9k fix another typo

# change to:
pick a1b2c3d add login form
squash d4e5f6g fix typo in login form
squash h7i8j9k fix another typo

# result: one clean commit "add login form" instead of 3 messy ones
```

```
  BEFORE SQUASH:                    AFTER SQUASH:

  [add form] [fix typo] [fix typo]  [add login form]
  (3 messy commits)                 (1 clean commit)

  Your PR reviewer sees ONE logical change,
  not your "oops, typo" history.
```

---

## Tags -- Mark Important Points

```bash
# create a lightweight tag
git tag v1.0.0

# create an annotated tag (recommended for releases)
git tag -a v1.0.0 -m "first production release"

# push tags to remote
git push origin v1.0.0
git push origin --tags    # push all tags

# list tags
git tag

# checkout a specific tag
git checkout v1.0.0
```

---

## Git Bisect -- Find Which Commit Broke Something

Binary search through commits to find the exact commit that introduced a bug.

```bash
# start bisect
git bisect start

# mark current commit as bad (bug exists)
git bisect bad

# mark a known good commit (bug didn't exist here)
git bisect good a1b2c3d

# Git checks out a commit in the middle. Test it, then:
git bisect good    # if the bug is NOT present
# or
git bisect bad     # if the bug IS present

# Git narrows down. Repeat until it finds the exact commit.
# "a1b2c3d is the first bad commit"

# done
git bisect reset
```

```
  BISECT = binary search through history

  100 commits between "good" and "bad"

  Step 1: test commit 50    -> bad
  Step 2: test commit 25    -> good
  Step 3: test commit 37    -> bad
  Step 4: test commit 31    -> good
  Step 5: test commit 34    -> bad
  Step 6: test commit 32    -> good
  Step 7: test commit 33    -> BAD! (this is the one!)

  Only 7 tests instead of 100. Finds the bug in O(log n).
```

---

## Useful Aliases

Add these to your `~/.gitconfig` for faster commands:

```bash
git config --global alias.s "status"
git config --global alias.co "checkout"
git config --global alias.br "branch"
git config --global alias.cm "commit -m"
git config --global alias.lg "log --oneline --graph --all"
git config --global alias.unstage "reset HEAD --"
git config --global alias.last "log -1 HEAD"
```

Now you can type:
```bash
git s          # instead of git status
git co main    # instead of git checkout main
git cm "msg"   # instead of git commit -m "msg"
git lg         # pretty log with branch graph
```

---

# Part C: Common Mistakes and Fixes

---

### "I committed to the wrong branch"

```bash
# move the last commit to the correct branch
git checkout correct-branch
git cherry-pick main           # copy the commit here
git checkout main
git reset --soft HEAD~1        # remove it from wrong branch
```

### "I need to undo my last commit but keep changes"

```bash
git reset --soft HEAD~1
# changes are back in staging area, ready to re-commit
```

### "I pushed sensitive data (API key, password)"

```bash
# remove the file from tracking (keeps local copy)
git rm --cached .env
echo ".env" >> .gitignore
git commit -m "remove .env from tracking"
git push

# WARNING: the secret is STILL in git history.
# You must rotate the key/password immediately.
# For full removal from history: use git filter-branch or BFG Repo Cleaner
```

### "I want to undo a pushed commit"

```bash
# SAFE way: revert (creates a new commit that undoes it)
git revert HEAD
git push
```

### "My branch is behind main and has conflicts"

```bash
# option 1: merge main into your branch
git checkout your-branch
git merge main
# resolve conflicts, then commit

# option 2: rebase on top of main (cleaner history)
git checkout your-branch
git rebase main
# resolve conflicts at each step, then:
git rebase --continue
```

---

## Quick Reference Card

| What you want | Command |
|---------------|---------|
| See what changed | `git status` |
| See line-by-line changes | `git diff` |
| Stage a file | `git add file.txt` |
| Commit | `git commit -m "message"` |
| Push | `git push origin branch` |
| Pull latest | `git pull origin main` |
| Create branch | `git checkout -b branch-name` |
| Switch branch | `git checkout branch-name` |
| Merge branch | `git merge branch-name` |
| Delete branch | `git branch -d branch-name` |
| Stash changes | `git stash` / `git stash pop` |
| Cherry-pick | `git cherry-pick commit-hash` |
| Rebase | `git rebase main` |
| Squash commits | `git rebase -i HEAD~N` |
| Undo last commit (keep changes) | `git reset --soft HEAD~1` |
| Undo last commit (delete changes) | `git reset --hard HEAD~1` |
| Undo a pushed commit | `git revert commit-hash` |
| Find who changed a line | `git blame file.txt` |
| Find which commit broke something | `git bisect start` |
| See all actions (safety net) | `git reflog` |
| Tag a release | `git tag -a v1.0.0 -m "release"` |
| See branch graph | `git log --oneline --graph --all` |

---

## Exercise Checklist

By the end of this class, every student should have:

- [ ] Forked the class-roster repo
- [ ] Cloned their fork locally
- [ ] Created a branch with their name
- [ ] Added their file in `students/`
- [ ] Committed with a clear message
- [ ] Pushed their branch to their fork
- [ ] Opened a Pull Request to the original repo
- [ ] (Bonus) Reviewed another student's PR and left a comment
