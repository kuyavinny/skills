---
name: npm-publish-cicd
description: Set up GitHub Actions CI/CD for an npm package with automated testing, security auditing, branch protection, and publish-on-tag workflow. Use when creating a new npm package repo or hardening an existing one for production publishing.
argument-hint: [repo path]
allowed-tools: Bash(git *), Bash(gh *), Read, Write, Edit
---

# npm Publish CI/CD Setup

You are tasked with setting up a complete CI/CD pipeline for publishing npm packages to the registry via GitHub Actions.

## Context

This skill configures a repo so that:
- Every push/PR runs build + audit
- Tag pushes (`v*`) trigger npm publish with provenance
- Branch protection requires PR review + passing CI
- Security hardening is baked in

## Process

### Step 0: Gather info

Ask the user for any missing information:

- **Repo path**: Local path to the git repo (required)
- **Package scope/name**: e.g., `@kuyavinny/pi-muninn-mem` (read from package.json if exists)
- **npm access**: `public` or `restricted` (scoped packages default to restricted)
- **Peer dependencies**: Any packages that must be listed as peerDeps
- **Build command**: e.g., `npx esbuild ...` or `npm run build` (read from package.json if exists)

### Step 1: Create CI workflow

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Verify bundle
        run: |
          test -f dist/index.mjs || (echo "❌ Bundle not found" && exit 1)
          SIZE=$(wc -c < dist/index.mjs)
          [ "$SIZE" -gt 1000 ] || (echo "❌ Bundle too small: $SIZE bytes" && exit 1)
          echo "✅ Bundle OK: $SIZE bytes"

      - name: Security audit
        run: npm audit --audit-level=high || true
        continue-on-error: true

  publish:
    needs: build
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/v')
    permissions:
      contents: read
      id-token: write

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
          registry-url: https://registry.npmjs.org

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Publish to npm with provenance
        run: npm publish --provenance --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

**Key points:**
- `tags: ['v*']` in push trigger enables publish on version tags
- `npm publish --provenance` links the package to the GitHub commit/build
- `--access public` required for scoped packages (they default to restricted)
- `NPM_TOKEN` must be added as a GitHub Actions secret
- The `dist/index.mjs` check should be adjusted to match your package's output file

### Step 2: Harden package.json

Ensure `package.json` has:

```json
{
  "scripts": {
    "build": "npx esbuild index.ts --bundle --format=esm --platform=node --outfile=dist/index.mjs --external:YOUR-PEER-DEPS",
    "prepublishOnly": "npm run build"
  },
  "peerDependencies": {
    "your-peer-dep": ">=MIN_VERSION"
  },
  "files": [
    "index.ts",
    "dist/",
    "src/",
    "README.md",
    "LICENSE",
    "package.json"
  ]
}
```

**Key points:**
- Use `npx esbuild` (not bare `esbuild`) — `esbuild` isn't in PATH during `npm publish`
- `prepublishOnly` ensures the bundle is built before publishing
- `files` whitelist prevents dev files from being published
- `peerDependencies` should be `>=MIN_VERSION`, not `*`
- Remove any dev-only files from `files` (AGENTS.md, Dockerfile, design docs)

### Step 3: Set up branch protection

```bash
gh api repos/{owner}/{repo}/branches/main/protection \
  --method PUT \
  --input - <<'EOF'
{
  "required_status_checks": {
    "strict": true,
    "contexts": ["build"]
  },
  "enforce_admins": true,
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "dismiss_stale_reviews": true
  },
  "restrictions": null
}
EOF
```

**Key points:**
- `enforce_admins: true` prevents even repo admins from pushing directly to main
- `dismiss_stale_reviews: true` resets approvals when new commits are pushed
- `strict: true` requires CI to pass on the merge commit, not just the branch head
- Temporarily set `enforce_admins: false` when merging PRs via CLI

### Step 4: Add NPM_TOKEN secret

```bash
gh secret set NPM_TOKEN --repo {owner}/{repo}
# paste the token when prompted
```

**Creating the token:**
1. Go to https://www.npmjs.com/settings/{username}/tokens
2. Generate New Token → **Granular Access Token**
3. Set: Packages = Read and write, Organizations = your scope
4. Copy the token and add it as a GitHub secret

**If you have 2FA enabled:**
- Granular tokens with "bypass 2FA for publishing" work for automation
- Classic Automation tokens also work and bypass 2FA
- For first-time publish, you may need to `npm publish --access public` manually with OTP, then the token works for subsequent publishes

### Step 5: Verify the setup

1. Push a change via PR:
   ```bash
   git checkout -b test/ci
   # make a small change
   git add -A && git commit -m "test: verify CI"
   git push -u origin test/ci
   gh pr create --title "test: verify CI" --body "Testing CI pipeline"
   # wait for CI to pass, then merge
   ```

2. Publish a version:
   ```bash
   npm version patch  # creates v1.0.x
   # merge version bump via PR
   git tag v1.0.x
   git push origin v1.0.x
   # watch: gh run list --repo {owner}/{repo}
   ```

### Step 6: Publishing workflow

For every new release:

```bash
# 1. Bump version
npm version patch  # or minor, major

# 2. Create PR, get review, merge
git checkout -b chore/v1.x.x
git push -u origin chore/v1.x.x
gh pr create --title "chore: bump to v1.x.x" --body "Version bump"
# merge via CLI or GitHub

# 3. Tag and push (triggers publish)
git tag v1.x.x
git push origin v1.x.x
```

## Troubleshooting

### `npm publish` fails with EOTP
- Your npm account has 2FA enabled. Either:
  - Use `npm publish --access public --otp=YOUR_CODE` for first publish
  - Create a Granular Access Token with "bypass 2FA for publishing"
  - Create a Classic Automation token (always bypasses 2FA)

### `esbuild: command not found` during `npm publish`
- Use `npx esbuild` in build scripts, not bare `esbuild`
- The `prepublishOnly` script runs in a context where `node_modules/.bin` may not be in PATH

### Tag push doesn't trigger publish
- Ensure `tags: ['v*']` is in the workflow's `on: push` section
- The tag must point to a commit that exists on the main branch
- Delete and re-create the tag if needed: `git tag -d v1.x.x && git tag v1.x.x && git push -f origin v1.x.x`

### Branch protection blocks direct pushes
- Create a feature branch, push, open PR, merge
- Temporarily disable `enforce_admins` for admin merges:
  ```bash
  gh api repos/{owner}/{repo}/branches/main/protection \
    --method PUT --input - <<'EOF'
  {"enforce_admins": false, "required_pull_request_reviews": null, "restrictions": null, "required_status_checks": {"strict": true, "contexts": ["build"]}}
  EOF
  # merge PR
  # re-enable:
  gh api repos/{owner}/{repo}/branches/main/protection \
    --method PUT --input - <<'EOF'
  {"enforce_admins": true, "required_pull_request_reviews": {"required_approving_review_count": 1, "dismiss_stale_reviews": true}, "restrictions": null, "required_status_checks": {"strict": true, "contexts": ["build"]}}
  EOF
  ```

### First publish of a scoped package
- Scoped packages default to `restricted` (private) on npm
- Use `npm publish --access public` for the first publish
- Subsequent publishes can use `npm publish` (access level is remembered)

## Security hardening checklist

Before publishing, verify:

- [ ] `files` in package.json is a whitelist (no dev files)
- [ ] No secrets, API keys, or tokens in source code
- [ ] `.npmignore` excludes dev files (or `files` whitelist is used)
- [ ] `peerDependencies` use `>=MIN_VERSION`, not `*`
- [ ] Build script uses `npx` (not bare command names)
- [ ] `prepublishOnly` runs the build
- [ ] No binaries in the package (check with `find . -size +1M -not -path '*/node_modules/*'`)
- [ ] `package-lock.json` is committed (reproducible builds)
- [ ] NPM_TOKEN is a GitHub Actions secret, not in code
- [ ] Branch protection is enabled on main
- [ ] `npm audit --audit-level=high` passes