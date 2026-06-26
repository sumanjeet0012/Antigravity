# unified-testing Repository Profile

Applied when the repository name contains `unified-testing`.

---

## Repository Purpose

`unified-testing` contains cross-implementation performance and interoperability
test definitions for libp2p implementations (go, rust, js, nim, py, etc.).
CI workflows detect which implementation changed and run only the affected tests.

---

## YAML Validity — CRITICAL

Any YAML file in the diff must be syntactically valid and fully parseable.

**Known failure mode:** Unresolved git conflict markers left in YAML files
(`<<<<<<<`, `=======`, `>>>>>>>`) silently break the `detect-changed-impls`
composite action, causing CI jobs to be skipped with no error message.

For every YAML file in the diff:
1. Confirm no conflict markers are present (`<<<<<<<`, `=======`, `>>>>>>>`).
2. Confirm the YAML structure is valid (correct indentation, no duplicate keys).
3. Pay special attention to `perf/images.yaml` — this file gates which
   implementation CI jobs run.

If conflict markers are found: report as **CRITICAL**.

---

## CI Workflow Changes

For any changes to `.github/workflows/`:

- Verify `on:` triggers match the intended scope (paths, branches).
- Verify `needs:` job dependencies are correct and complete.
- Verify matrix configurations produce at least one job (empty matrix = silent skip).
- Verify `detect-changed-impls` outputs are correctly consumed by downstream jobs.
- Check that no job is conditionally skipped in a way that would hide real failures.

---

## Implementation Directory Changes

For changes to implementation directories (e.g., `implementations/py/`,
`perf/impl/py-libp2p/`):

- Implementation name in the directory must match the key in `perf/images.yaml`
  (or equivalent registry file). Mismatch = silent CI skip.
- Dockerfile syntax must be valid.
- Entrypoint scripts must be executable (`chmod +x`) and syntactically correct.
- Version pinning of dependencies must be justified in the PR description.

---

## Test Definition Changes

For changes to test definition files (JSON, YAML, TOML test specs):

- All referenced implementation names must exist in the images registry.
- Parameter ranges must be valid (no negative sizes, no zero timeouts).
- Verify no existing test case is accidentally removed.
- If adding a new test: verify it is reachable by the CI matrix.

---

## Python Implementation

If changes touch a Python implementation directory, additionally apply:
- `rules/python.md`
- `rules/async-python.md`

---

## README and Documentation

- Adding a new implementation → README must be updated with the implementation entry.
- Changing test parameters → any existing documentation of expected results
  must be updated.

---

## Security

- No secrets, API keys, or credentials in any file.
- Container images must pull from pinned digests or locked tags, not `latest`.
- Build scripts must not `curl | bash` from unverified sources.
