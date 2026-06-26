# Generic Repository Profile

Applied when no specific profile is detected for the repository.

---

## Changelog and Release Notes

- Check for `CHANGELOG.md` or `HISTORY.md` updates when user-facing behavior changes.
- Breaking changes must have a migration note or upgrade guide.
- New public APIs must appear in the changelog.

## Newsfragments (towncrier)

If a `newsfragments/` directory exists in the repository:

**Format:** `<ISSUE_OR_PR_NUMBER>.<TYPE>.rst`

Valid types: `breaking`, `bugfix`, `deprecation`, `docs`, `feature`, `internal`, `misc`, `performance`, `removal`

Rules:
- One file per linked issue (not one per PR).
- If the PR fixes multiple issues, each issue needs its own newsfragment.
- Content must describe the change from a **user perspective**, not a developer perspective.
- File must end with a newline character.

If missing or malformed: report as **CRITICAL — BLOCKER**.

## CI Checks

- All existing CI checks must pass.
- New code must not disable or skip existing CI checks without justification.
- If CI is failing before this PR, note it but do not block the PR for pre-existing failures.

## General Rules

- Public APIs must have docstrings or comments.
- Breaking changes must be explicitly documented.
- Tests must be added for all new functionality.
- Error cases must be covered in tests, not just the happy path.
- No secrets, credentials, or tokens in any committed file.
