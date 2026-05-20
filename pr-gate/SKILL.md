# PR Gate Skill

Enforce the mandatory PR workflow before and after every PR implementation.

## When to invoke

Invoke this skill **at the start of every PR implementation** and **after every PR is completed**.

## Pre-PR checklist

Before writing any code:

- [ ] Feature branch exists, branched from `develop`
- [ ] Todo list is built, scoped to **this PR only** (no future PRs)
- [ ] Scope is clear — what is and is not in this PR
- [ ] User has approved the scope

## Post-PR checklist (STOP here — do not continue automatically)

After completing a PR:

- [ ] All tests pass (unit + integration)
- [ ] Code review has been run (`/code-review` skill) and findings fixed
- [ ] PR merged into `develop`
- [ ] **STOP — do not begin the next PR**
- [ ] Report: what was done, test counts, branch state
- [ ] Wait for explicit "next", "continue", or "PR X" before proceeding

## Key rules

1. **One PR at a time** — no chaining without explicit instruction
2. **Code review is mandatory** — run `/code-review` before every merge
3. **Stop and report** — after every PR completion, stop and present results
4. **"Continue implementing" means this PR only** — do not assume

## Violations

If you catch yourself about to:
- Start a second PR without being asked
- Skip a code review
- Merge without user approval

...stop and ask instead.