# Workflow Quick Reference

**Model:** Contractor (you direct, Claude orchestrates)

---

## The Loop

```
Your instruction
    ↓
[PLAN] (if multi-file or unclear) → Show plan → Your approval
    ↓
[EXECUTE] Implement, verify, done
    ↓
[REPORT] Summary + what's ready
    ↓
Repeat
```

---

## I Ask You When

- **Design forks:** "Option A (fast) vs. Option B (robust). Which?"
- **Code ambiguity:** "Spec unclear on X. Assume Y?"
- **Replication edge case:** "Just missed tolerance. Investigate?"
- **Scope question:** "Also refactor Y while here, or focus on X?"

---

## I Just Execute When

- Code fix is obvious (bug, pattern application)
- Verification (tolerance checks, tests, compilation)
- Documentation (logs, commits)
- Plotting (per established standards)
- Deployment (after you approve, I ship automatically)

---

## Quality Gates (No Exceptions)

| Score | Action |
|-------|--------|
| >= 80 | Ready to commit |
| < 80  | Fix blocking issues |

---

## Non-Negotiables

- **Path convention:** All paths relative to project root — never hardcode `/Users/hj/...`
- **Seed convention:** `set seed 20230101` in STATA; `rng(20230101, 'twister')` in MATLAB
- **Figure standards:** AEA style — PDF vector output, no gridlines, no in-figure title, fonts ≥ 8pt, width ≤ 6.5in (full-page) or 3.5in (half-page); caption goes in LaTeX
- **Tolerance thresholds:** GE convergence 1e-8 (MATLAB fixed-point); STATA coefficient precision 1e-6; welfare changes reported to 1e-4

---

## Preferences

**Visual:** AEA style — clean, minimal, publication-ready from first draft
**Reporting:** Concise bullets; prose only when explaining model derivations
**Session logs:** Always (post-plan, incremental, end-of-session)
**Replication:** Strict — flag near-misses even within tolerance; investigate before closing

---

## Exploration Mode

For experimental work, use the **Fast-Track** workflow:
- Work in `explorations/` folder
- 60/100 quality threshold (vs. 80/100 for production)
- No plan needed — just a research value check (2 min)
- See `.claude/rules/exploration-fast-track.md`

---

## Next Step

You provide task → I plan (if needed) → Your approval → Execute → Done.
