## Checking Out Branches

### List branches
```bash
git branch  # List local branches
git branch -r  # List remote branches
git branch -a  # List all branches (local and remote)
```

### Create and switch to a new branch
```bash
git checkout -b new-branch-name
```

### Switch to an existing branch
```bash
git checkout branch-name
```

### Create a new branch from a specific commit
```bash
git checkout -b new-branch-name commit-hash
```

### Checkout a remote branch
```bash
git checkout -b local-branch-name origin/remote-branch-name
```

## Merging Branches

### Merge a branch into the current branch
```bash
git merge branch-name
```

### Merge with a commit message
```bash
git merge --no-ff -m "Merge message" branch-name
```

### Abort a merge in progress
```bash
git merge --abort
```

### Merge specific files from another branch
```bash
git checkout branch-name -- path/to/file1 path/to/file2
```

## Handling Merge Conflicts

1. Open conflicted files and resolve conflicts manually
2. Stage resolved files:
   ```bash
   git add path/to/resolved-file
   ```
3. Complete the merge:
   ```bash
   git commit
   ```

## Key Points and Gotchas

- Always ensure your working directory is clean before checking out or merging
- Use `git status` to check the state of your working directory
- `git fetch` before merging to ensure you have the latest remote changes
- Consider using `git merge --no-ff` to preserve feature branch history
- Be cautious when force-pushing (`git push -f`) after changing branch history
- Use `git log --graph --oneline --all` to visualize branch structure

Remember: Regular commits, clear commit messages, and frequent pushes to remote repositories help maintain a clean and manageable Git history.