# Deploy Workflow — Detailed Steps

## Prerequisites
- Git repository initialized with remote
- Component created in `components/<name>/`
- `.limio.json` configured with valid credentials

## Smart Deploy Steps

The deploy endpoint (`/api/deploy`) performs these steps:

### 1. Validate Component
```bash
# Check component folder exists
ls components/<component-name>/
```

### 2. Check Git State
```bash
git rev-parse --abbrev-ref HEAD    # Get current branch
git rev-parse --abbrev-ref <branch>@{upstream}  # Check tracking
```

### 3. Fetch Remote
```bash
git fetch origin
```

### 4. Check Ahead/Behind
```bash
git rev-list --left-right --count <branch>...origin/<branch>
```

### 5. If Behind Remote — Pull with Rebase
```bash
# Stash uncommitted changes if needed
git stash push -m "deploy-auto-stash" --include-untracked

# Pull with rebase
git pull --rebase origin <branch>

# Pop stash
git stash pop
```

If rebase conflicts: abort rebase, pop stash, return error to user.
If stash pop conflicts: return error, user must resolve manually.

### 6. Stage Files
```bash
git add components/<component-name>/

# Also stage related story files
git add component-playground/src/stories/<related-stories>.js
```

### 7. Check for Changes
```bash
git diff --cached --name-only
```
If empty, report "No changes to deploy — already up to date"

### 8. Commit and Push
```bash
git commit -m "Deploy component: <component-name>"
git push  # or git push -u origin <branch> if no upstream
```

If push fails, retry once:
```bash
git pull --rebase origin <branch>
git push
```

### 9. Monitor Build
Poll `<limio-base-url>/api/component/builds` for build status.
Only match builds that started after the deploy timestamp.

## Deploy Status States

| State | Progress | Meaning |
|-------|----------|---------|
| pushing | 0-40% | Git commit and push |
| building | 40-90% | Limio CI/CD building |
| success | 100% | Build completed |
| error | - | Something failed |
| timeout | - | Build took too long |

## Error Recovery

### Merge Conflict During Pull
1. `git rebase --abort`
2. `git stash pop` (if stashed)
3. User must resolve conflicts manually
4. Re-run deploy after resolution

### Push Rejected
1. Pull with rebase
2. Retry push
3. If still fails, user must resolve manually

### Build Failed
1. Check Limio admin panel for build logs
2. Common issues: syntax errors, missing deps, invalid imports
3. Fix component and re-deploy
