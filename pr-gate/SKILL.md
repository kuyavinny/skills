# PR Gate Skill

Enforce the mandatory PR workflow before and after every PR implementation.

## When to invoke

Invoke this skill **at the start of every PR implementation** and **after every PR is completed**.

## Branch naming conventions

Branch names adapt to the project's structure:

| Project structure | Pattern | Example |
|-------------------|---------|--------|
| Modularized (modules/epics tracked by number) | `feat/mod{M}-pr{N}-{description}` | `feat/mod3-pr5-criteria-evaluation` |
| Non-modular (feature-based PRs) | `feat/pr{N}-{description}` | `feat/pr2-auth-middleware` |
| Issue-linked (has ticket IDs) | `feat/{ticket}-{description}` | `feat/PROJ-123-add-ldap` |

**Convention:** At the start of each PR, derive the branch name from whichever pattern matches the project, then use that same `{branch_name}` consistently in the post-PR report template.

## PR lifecycle

```
1. Advisor cross-check  →  Verify PR scope against tech spec + PRD before coding
2. Implement            →  Write code, scoped to this PR only
3. code-review skill    →  Audit the diff, write review artifact
4. validate skill       →  Verify plan success criteria met in working tree
5. PR + merge           →  Push branch, create PR, merge to develop
6. Report               →  Fill post-pr-report.md template
```

### Step 1: Advisor cross-check (pre-implementation)

Before writing any code, call `advisor()` to cross-check the PR's scope against:
- The implementation plan phase description
- The technical specification
- The PRD

This serves as the plan review gate. If the advisor flags misalignment, resolve before proceeding.

### Step 2: Implement

- Feature branch must exist, branched from `develop`.
- Todo list scoped to **this PR only** (no future PRs).
- Scope must be clear — what is and is not in this PR.

### Step 3: code-review skill

- Run the `code-review` skill (`/skill:code-review`) on the PR diff.
- Review artifact MUST be written to `.rpiv/artifacts/reviews/`.
- Review artifact MUST contain `status: approved`.
- If any findings exist, fix them and re-run `code-review` until approved.
- No self-review shortcuts.

### Step 4: validate skill

- Run the `validate` skill (`/skill:validate`) after code-review passes.
- This verifies that the plan's success criteria are actually met in the working tree.
- If validation fails, fix and re-run.

### Step 5: PR + merge

- Push branch to origin.
- Create PR targeting `develop`.
- Merge PR.
- Delete branch.

### Step 6: Report

- Fill `templates/post-pr-report.md`.
- Report: what was done, test counts, branch state.

## Key rules

1. **One PR at a time** — no chaining without explicit instruction (unless user grants batch mode).
2. **Advisor cross-check before coding** — every PR starts with advisor verifying scope against specs.
3. **Code review is mandatory** — run the `code-review` skill. Fix all findings until approved.
4. **Validation is mandatory** — run the `validate` skill after code-review passes.
5. **Stop and report** — after every PR completion, stop and present results (unless batch mode).
6. **"Continue implementing" means this PR only** — do not assume.

## Batch mode

When the user explicitly grants batch mode (e.g., "complete all remaining PRs"):
- Skip the STOP between PRs — continue automatically to the next PR.
- All other rules still apply: advisor, code-review, validate, report per PR.
- Batch mode ends after the current session; default behavior reverts next session.

## Violations

If you catch yourself about to:
- Start a second PR without being asked (outside batch mode)
- Skip the advisor cross-check
- Skip code review
- Skip validation
- Merge without review approval

...stop and ask instead.