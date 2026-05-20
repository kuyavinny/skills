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

## Pre-PR checklist

Before writing any code:

- [ ] Feature branch exists, branched from `develop`
- [ ] Todo list is built, scoped to **this PR only** (no future PRs)
- [ ] Scope is clear — what is and is not in this PR
- [ ] User has approved the scope

## Post-PR checklist (STOP here — do not continue automatically)

After completing a PR:

- [ ] All tests pass (unit + integration)
- [ ] **Code review is mandatory** — run the `code-review` skill (`/skill:code-review`) on the PR
  * Review artifact MUST be written to `.rpiv/artifacts/reviews/`
  * Review artifact MUST contain `status: approved`
  * If any findings exist, fix them and re-run `code-review` until approved
- [ ] PR merged into `develop`
- [ ] **STOP — do not begin the next PR**
- [ ] Report: what was done, test counts, branch state
- [ ] Wait for explicit "next", "continue", or "PR X" before proceeding

## Key rules

1. **One PR at a time** — no chaining without explicit instruction
2. **Code review is mandatory** — run the `code-review` skill (`/skill:code-review`) on the PR branch. Review artifact must exist at `.rpiv/artifacts/reviews/` with `status: approved`. Fix all findings and re-run until approved. No self-review shortcuts.
3. **Stop and report** — after every PR completion, stop and present results
4. **"Continue implementing" means this PR only** — do not assume

## Violations

If you catch yourself about to:
- Start a second PR without being asked
- Skip a code review
- Merge without user approval

...stop and ask instead.