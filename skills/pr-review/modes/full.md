# Full Review Mode

Comprehensive review. This is the default mode.

---

## What to Run

- All seven core review areas.
- Full repository profile (including build/linting commands where specified).
- All language rules.
- All linked issues read in full (body + comments).
- All PR review comments read.

---

## What to Report

All report sections. No omissions.
Write "No issues found." for any section with no findings.

---

## py-libp2p specific (full mode)

Run validation commands after checking out the PR branch:

```bash
source venv/bin/activate
make pr
```

If documentation-related files changed:

```bash
make linux-docs
```

Report the result of each command in section 10 (Testing Review) of the report.
