---
name: git-flow
description: Strict Git Flow branching model. Mandatory for all git operations — branching, merging, PRs, releases, and hotfixes. Invoke before any git merge, push, or branch creation.
---

# Git Flow Skill

This skill enforces a strict Git Flow branching model. **Every git operation must follow these rules.** Violations are blocking errors.

## Branch Layout

```
main        ← production-ready. Only updated during releases and hotfixes. NEVER merge features here.
 ├── develop ← integration branch. All feature branches merge here.
 │    ├── feat/*     ← feature branches (branch off develop, merge back to develop)
 │    ├── fix/*      ← bugfix branches (branch off develop, merge back to develop)
 │    └── refactor/* ← refactor branches (branch off develop, merge back to develop)
 └── hotfix/*  ← hotfix branches (branch off main, merge to both main AND develop)
```

## Golden Rules

### Rule 1: `main` is sacred

**Never merge into `main` unless performing a release or hotfix.**

`main` tracks production. It is only updated:
1. During a **release** (via `release/*` branch)
2. During a **hotfix** (via `hotfix/*` branch)

If you catch yourself about to push or merge to `main` from anything other than a release or hotfix, **stop immediately**.

### Rule 2: Feature branches merge into `develop`

All feature, fix, and refactor branches:
1. Branch off `develop`
2. Get reviewed via PR targeting `develop`
3. Merge into `develop` after approval

### Rule 3: PRs target `develop`, not `main`

When creating a PR, always set `--base develop`:

```bash
gh pr create --base develop --head feat/my-feature --title "..." --body "..."
```

### Rule 4: Releases branch off `develop`, merge into `main`

When cutting a release:
1. Create a `release/vX.Y.Z` branch from `develop`
2. Apply release-specific fixes on the release branch
3. Merge `release/vX.Y.Z` into `main` (and tag)
4. Merge `release/vX.Y.Z` back into `develop`
5. Delete the release branch

### Rule 5: Hotfixes branch off `main`, merge into both

When hotfixing production:
1. Create `hotfix/vX.Y.Z` from `main`
2. Fix and test
3. Merge `hotfix/vX.Y.Z` into `main` (and tag)
4. Merge `hotfix/vX.Y.Z` into `develop`
5. Delete the hotfix branch

## Checklist (invoke on every git operation)

Before any merge or push, run this checklist:

- [ ] **Am I touching `main`?** → Only allowed for releases and hotfixes. If not, target `develop`.
- [ ] **Is my PR base `develop`?** → Must be. Never `--base main` for feature work.
- [ ] **Am I branching from `develop`?** → All feature branches start from `develop`.
- [ ] **After merge, did I pull `develop`?** → Always `git pull origin develop` after a merge.
- [ ] **Do I need a release?** → If the user says "release" or "push to production", use the release flow. Otherwise, stay on `develop`.

## Common Commands

### Start a feature

```bash
git checkout develop && git pull origin develop
git checkout -b feat/my-feature
```

### Finish a feature (via PR)

```bash
git push -u origin feat/my-feature
gh pr create --base develop --head feat/my-feature --title "feat: description"
# After approval and merge:
git checkout develop && git pull origin develop
git branch -d feat/my-feature  # if not auto-deleted
```

### Start a release

```bash
git checkout develop && git pull origin develop
git checkout -b release/v1.2.0
# Apply release-specific fixes if needed
# Then:
git checkout main && git merge release/v1.2.0
git tag -a v1.2.0 -m "Release v1.2.0"
git checkout develop && git merge release/v1.2.0
git push origin main develop --tags
git branch -d release/v1.2.0
```

### Start a hotfix

```bash
git checkout main && git pull origin main
git checkout -b hotfix/v1.2.1
# Fix and test
git checkout main && git merge hotfix/v1.2.1
git tag -a v1.2.1 -m "Hotfix v1.2.1"
git checkout develop && git merge hotfix/v1.2.1
git push origin main develop --tags
git branch -d hotfix/v1.2.1
```

## Error Recovery

If you accidentally merged into `main`:
1. **Stop.** Do not push further.
2. Reset `main` to its correct state: `git update-ref refs/heads/main <correct-commit> && git push origin main --force`
3. Merge the feature correctly into `develop` instead.
4. Record the mistake in memory to avoid recurrence.

## Activation

This skill activates automatically for any git operation involving:
- `git merge`
- `git push`
- `gh pr create`
- Branch creation (`git checkout -b`)
- Any workflow involving `main`, `develop`, or release branching

When in doubt, **target `develop` — never `main`.**