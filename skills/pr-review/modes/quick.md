# Quick Review Mode

Fast, high-signal review. Focus on blockers only.

---

## What to Run

- Core review: Correctness and Security only.
- Apply repository profile: newsfragment check ONLY (skip build/linting runs).
- Apply language rules: CRITICAL items only.
- Skip: Architecture deep-dive, Performance profiling, Documentation build.

---

## What to Report

Include in the report:
- Critical issues (all)
- Major issues (all)
- Newsfragment validation result
- Merge readiness verdict
- Security impact

Omit:
- Minor issues
- Performance notes
- Documentation suggestions
- Questions for the author (unless directly related to a critical issue)

---

## Time Target

Single pass through the diff. Do not read every linked issue comment thread
unless a critical issue depends on that context.
