# Git Sync

Synchronize the current branch with the latest changes from the main/master branch.

## Usage

```
/git/sync
```

## What this does

1. Fetches the latest changes from the remote repository
2. Identifies the default branch (main or master)
3. Rebases the current branch on top of the default branch
4. Pushes the updated branch to the remote (with force-with-lease for safety)

## Steps

```bash
# Fetch latest remote changes
git fetch origin

# Determine default branch
DEFAULT_BRANCH=$(git remote show origin | grep 'HEAD branch' | awk '{print $NF}')

# Rebase current branch onto default branch
git rebase origin/$DEFAULT_BRANCH

# Push updated branch (safe force push)
git push --force-with-lease
```

## Notes

- Uses `--force-with-lease` instead of `--force` to avoid overwriting remote changes made by others
- If rebase conflicts occur, resolve them manually and run `git rebase --continue`
- To abort a failed rebase, run `git rebase --abort`
- Do **not** use this on shared branches (e.g., main, develop) — only on feature branches
