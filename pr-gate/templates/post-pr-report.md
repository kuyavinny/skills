---
pr_number:       {N}
branch:          {branch_name}
merge_commit:    {hash}
status:          merged
author:          {author}
repository:      {repo}
base_branch:     develop
code_review:     .rpiv/artifacts/reviews/{datetime}_{branch_slug}.md
review_status:   approved
---

# PR {N}: {Title}

**Merged by {author}** at `{merge_commit}` into `{base_branch}`

## 1. What Was Done

{Brief paragraph describing the scope of changes}

---

## 2. Files Changed

| File | Δ Lines | Purpose |
|------|---------|---------|
| `src/{file}.rs` | +{lines} | {One-line description} |
| `tests/{file}.rs` | +{lines} | {One-line description} |

### Summary
- **Files created:** {N}
- **Files modified:** {N}
- **Files deleted:** {N}
- **Net lines added:** +{N} / −{N}

---

## 3. Design Decisions

| Decision | Rationale |
|----------|-----------|
| {Decision #1} | {Why this choice was made} |
| {Decision #2} | {Why this choice was made} |

---

## 4. Test Results

| Suite | Count | Status |
|-------|-------|--------|
| Unit tests (lib) | {N} | ✅ passing |
| Integration tests | {N} | ✅ passing |
| Workspace total | {N} | ✅ passing |
| **Clippy** | — | ✅ clean / 🔴 warnings |

*No tests added / modified for this PR* ← when applicable

---

## 5. Code Review

- **Review artifact:** `{review_path}`
- **Status:** `{approved|changes_requested|pending}`
- **Findings:** 🔴 {critical} · 🟡 {important} · 🔵 {suggestions} · ✅ {fixed}
- **Review type:** `{automated|manual|peer}`

{Optional: brief note on any notable findings and how they were addressed}

---

## 6. Boundary / Security Notes

{Optional section for boundary enforcement, security considerations, or notable
architectural constraints verified in this PR}

---

## 7. Next Steps

**⚠️ STOP — Waiting for explicit instruction before proceeding**

To continue, say one of:
- **"Implement PR {N+1}"** or **"Continue"** — Begin next PR
- **"Show plan"** — Review the remaining phases
- Any other direction

---

## Appendix: Branch State

```
{git log --oneline -3 output}
```
