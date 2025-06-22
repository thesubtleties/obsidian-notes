## 🚀 Essential Setup (Do This First!)

```bash
# Set your identity (required for commits)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Optional but recommended
git config --global init.defaultBranch main
git config --global core.editor "code --wait"  # VS Code as editor
```

**Why this matters:** Git tracks who made each change. Without this setup, your commits will be attributed to generic system info.

---

## 📁 Starting a Repository

| Command | What It Does | When To Use |
|---------|-------------|-------------|
| `git init` | Creates new repo in current folder | Starting a new project |
| `git clone <url>` | Downloads existing repo | Working on existing project |
| `git clone <url> <folder-name>` | Downloads repo to specific folder | Want custom folder name |

---

## 📊 Checking Status & History

```bash
# The most important command - use it constantly!
git status

# See what changed in files
git diff                    # Unstaged changes
git diff --staged          # Staged changes
git diff HEAD~1            # Compare to previous commit

# View commit history
git log                    # Full history
git log --oneline          # Compact view
git log --graph --oneline  # Visual branch structure
```

**Pro tip:** `git status` is your best friend. Run it before and after every operation until Git becomes second nature.

---

## 🎯 The Core Workflow (90% of your Git usage)

### 1. Stage Changes
```bash
git add <filename>         # Add specific file
git add .                  # Add all changes in current directory
git add -A                 # Add all changes in entire repo
```

### 2. Commit Changes
```bash
git commit -m "Your commit message"
git commit -am "Message"   # Add + commit tracked files in one step
```

### 3. Push to Remote
```bash
git push                   # Push current branch
git push origin main       # Push specific branch
```

**The staging concept:** Git has a "staging area" where you prepare changes before committing. This lets you commit only specific changes, even from the same file.

---

## 🌿 Branch Management

| Command | Purpose | Example |
|---------|---------|---------|
| `git branch` | List branches | See all local branches |
| `git branch <name>` | Create branch | `git branch feature-login` |
| `git checkout <name>` | Switch branch | `git checkout main` |
| `git checkout -b <name>` | Create + switch | `git checkout -b bug-fix` |
| `git switch <name>` | Switch branch (newer) | `git switch main` |
| `git switch -c <name>` | Create + switch (newer) | `git switch -c new-feature` |

```bash
# Delete branches
git branch -d <name>       # Safe delete (merged only)
git branch -D <name>       # Force delete (careful!)
```

**Why branches matter:** Branches let you work on features without breaking main code. Think of them as parallel universes for your code.

---

## 🔄 Working with Remotes

```bash
# See remote repositories
git remote -v

# Add remote (usually done once)
git remote add origin <url>

# Fetch latest info (doesn't change your files)
git fetch

# Pull latest changes (fetch + merge)
git pull                   # Pull current branch
git pull origin main       # Pull specific branch

# Push your changes
git push origin <branch>   # First time pushing new branch
git push                   # Subsequent pushes
```

**Fetch vs Pull:** `fetch` downloads info about changes but doesn't apply them. `pull` downloads AND applies changes to your current branch.

---

## 🔧 Fixing Common Mistakes

### Undo Changes

| Scenario | Command | What It Does |
|----------|---------|-------------|
| Unstaged file changes | `git checkout -- <file>` | Discard changes to file |
| All unstaged changes | `git checkout -- .` | Discard all unstaged changes |
| Staged changes | `git reset HEAD <file>` | Unstage file (keeps changes) |
| Last commit (not pushed) | `git reset --soft HEAD~1` | Undo commit, keep changes staged |
| Last commit + changes | `git reset --hard HEAD~1` | ⚠️ Completely undo commit |

### Fix Commit Messages
```bash
git commit --amend -m "New message"  # Fix last commit message
git commit --amend --no-edit         # Add changes to last commit
```

**⚠️ Warning:** Never use `--hard` reset or `--amend` on commits you've already pushed to shared repositories!

---

## 🔍 Useful Investigation Commands

```bash
# Who changed what?
git blame <filename>       # See who wrote each line

# Find specific changes
git log --grep="bug"       # Search commit messages
git log -p <filename>      # See all changes to a file
git log --author="John"    # See commits by specific person

# See what's in a commit
git show <commit-hash>     # Show specific commit
git show HEAD~2           # Show commit 2 steps back
```

---

## 🆘 Emergency Commands

```bash
# Stash changes temporarily
git stash                  # Save current changes
git stash pop             # Restore stashed changes
git stash list            # See all stashes

# Get out of merge conflicts
git merge --abort         # Cancel merge
git rebase --abort        # Cancel rebase

# See where you are
git log --oneline -10     # Last 10 commits
git branch -a             # All branches (local + remote)
```

**Stashing is a lifesaver:** When you need to quickly switch branches but aren't ready to commit, stash saves your work temporarily.

---

## 📝 Commit Message Best Practices

### Good Format:
```
Add user authentication system

- Implement login/logout functionality
- Add password hashing with bcrypt
- Create user session management
- Add input validation for forms
```

### Quick Reference:
- **First line:** Brief summary (50 chars max)
- **Blank line:** Separates summary from details
- **Body:** Explain what and why, not how
- **Use imperative mood:** "Add feature" not "Added feature"

---

## 🎯 Daily Workflow Checklist

1. **Start of day:** `git pull` (get latest changes)
2. **Before working:** `git status` (check current state)
3. **While working:** `git add` → `git commit` (save progress frequently)
4. **End of day:** `git push` (backup your work)
5. **Before switching tasks:** `git status` (ensure clean state)

---

## 🚨 Red Flags - When to Ask for Help

- Merge conflicts you don't understand
- Accidentally committed sensitive data (passwords, keys)
- Repository history looks broken
- Can't push and getting cryptic errors
- Accidentally deleted important commits

**Remember:** Git is designed to be safe. Most "disasters" are recoverable with the right commands. When in doubt, ask before experimenting!

---

## 🔗 Essential Resources

- **Official Git Documentation:** [git-scm.com](https://git-scm.com)
- **Interactive Tutorial:** [learngitbranching.js.org](https://learngitbranching.js.org)
- **Visual Git Reference:** [ndpsoftware.com/git-cheatsheet.html](https://ndpsoftware.com/git-cheatsheet.html)

**Pro tip:** Keep this cheatsheet handy and refer to it often. Git mastery comes from repetition and understanding the core concepts, not memorizing every command.