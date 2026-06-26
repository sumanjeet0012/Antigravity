---
name: pr-review
description: >
  Use this skill when the user asks to review a pull request. Triggers:
  "Review PR #N", "Quick review of PR #N", "Security review of PR #N",
  "Architecture review of PR #N", "Performance review of PR #N".
  Repository-aware: auto-detects the repo, loads the right profile and
  language rules, runs the correct review mode, and saves a structured report.
compatibility: "antigravity, Claude Desktop, Cowork — any surface with GitHub MCP or GitHub CLI access"
license: MIT
---

# PR Review Skill

Production-quality, repository-aware pull request review for any codebase.

---

## Trigger Phrases

| User Input | Mode Detected |
|---|---|
| "Review PR #1029" | full (default) |
| "Quick review of PR #1029" | quick |
| "Security review of PR #1029" | security |
| "Architecture review of PR #1029" | architecture |
| "Performance review of PR #1029" | performance |
| "Full review of PR #1029" | full |

---

## Execution Workflow

Run these phases **in strict order**. Do not skip phases.

---

### Phase 1 — Parse Intent

Extract from the user's message:

- `PR_NUMBER` — required, numeric. Ask if missing before proceeding.
- `REPO` — from current working directory or user input.
- `MODE` — one of: `quick | full | security | architecture | performance`. Default: `full`.

---

### Phase 2 — Collect Context

**Prefer GitHub MCP. Fall back to GitHub CLI only if MCP fails or is unavailable.**

Collect in this order:

1. PR metadata: title, description, author, labels, assignees, reviewers
2. PR status and CI check results
3. PR review comments and threads
4. Changed files list
5. Full PR diff
6. Linked issues — scan PR body and commit messages for `Fixes #`, `Closes #`, `Resolves #`
7. For each linked issue: read issue body and all comments

Save all collected data. Reference it throughout the review.
Never fabricate data that was not collected.

**GitHub MCP tool calls:**
```
get_pull_request(PR_NUMBER)
list_pull_request_files(PR_NUMBER)
get_pull_request_diff(PR_NUMBER)
list_issue_comments(PR_NUMBER)
get_issue(ISSUE_NUMBER) — for each linked issue
list_issue_comments(ISSUE_NUMBER) — for each linked issue
```

**GitHub CLI fallback:**
```bash
gh pr view <PR_NUMBER>
gh pr view <PR_NUMBER> --comments
gh pr diff <PR_NUMBER>
gh issue view <ISSUE_NUMBER>
gh issue view <ISSUE_NUMBER> --comments
```

---

### Phase 3 — Detect Repository and Load Profile

Inspect the repository root for these manifest files (in order):

| File Found | Language / Ecosystem |
|---|---|
| `pyproject.toml` or `setup.py` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `package.json` | JavaScript / TypeScript |

Extract the project name from the detected manifest.

**Load repository profile:**

| Project Name Contains | Profile to Load |
|---|---|
| `py-libp2p` | `profiles/py-libp2p.md` |
| `unified-testing` | `profiles/unified-testing.md` |
| anything else | `profiles/generic.md` |

**Load language rules:**

| Language Detected | Rules to Load |
|---|---|
| Python | `rules/python.md` AND `rules/async-python.md` |
| Go | `rules/go.md` |
| JavaScript / TypeScript | `rules/javascript.md` |
| Rust | Use generic correctness rules (no Rust rules file yet) |

**Load review mode:**

| MODE | File to Load |
|---|---|
| quick | `modes/quick.md` |
| full | `modes/full.md` |
| security | `modes/security.md` |
| architecture | `modes/architecture.md` |
| performance | `modes/performance.md` |

---

### Phase 4 — Run Review Engine

Always run **all** of these core review areas, regardless of mode:

1. **Correctness** — logic errors, regressions, unawaited coroutines, race conditions
2. **Maintainability** — naming, responsibilities, code duplication, readability
3. **Architecture** — layer boundaries, coupling, cohesion, extension points
4. **Security** — input validation, auth, crypto, secrets, attack surface
5. **Performance** — allocations, blocking IO, unbounded growth, async efficiency
6. **Testing** — coverage of new logic, error cases, async test correctness
7. **Documentation** — docstrings, README, API references, usage examples

Then apply the **mode-specific focus** loaded in Phase 3.
Then apply the **repository profile rules** loaded in Phase 3.
Then apply the **language rules** loaded in Phase 3.

**Every finding must include:**
- Severity: `critical | major | minor`
- Confidence: `high | medium | low`
- File path and line reference (when available from the diff)
- Specific suggested fix
- Rationale (why this matters)

**Global rules:**
- Never fabricate code or findings not present in the diff.
- When uncertain, lower confidence rather than omit the finding.
- If linked issues exist, explicitly verify whether the PR resolves them.
- Every finding must have an actionable suggestion.
- Ignore cosmetic whitespace or comment formatting unless it would fail CI.

---

### Phase 5 — Generate Report

Load `templates/report.md` and fill every section.
Do not leave sections empty — write `"No issues found."` where applicable.

---

### Phase 6 — Save Report

Save the completed report to:
```
downloads/AI-PR-REVIEWS/<PR_NUMBER>-AI-PR-REVIEW.md
```

Create the directory if it does not exist.
Confirm the save path to the user after saving.
