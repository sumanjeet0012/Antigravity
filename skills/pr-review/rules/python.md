# Python Review Rules

Applied to all Python repositories.

---

## Type Hints

- All public functions and methods must have parameter and return type annotations.
- `-> None` is required on functions that return nothing.
- `Optional[X]` preferred over `X | None` unless the project targets Python 3.10+.
- `Any` usage must be accompanied by a comment explaining why it cannot be narrowed.
- `cast()` usage must be justified — it suppresses type errors without runtime checks.

---

## Exceptions

- Custom exceptions must subclass an appropriate stdlib base
  (`ValueError`, `RuntimeError`, `OSError`, etc.), not bare `Exception`.
- `except Exception` without re-raise is a major issue.
- Bare `except:` (catching `BaseException`) is a critical issue.
- Exception messages must be descriptive — "operation failed" is not acceptable.
- `__init__` must not leave the object in a partially initialized state on error.
- Never use exceptions for normal control flow in performance-sensitive paths.

---

## Imports

- Import order: stdlib → third-party → local (enforced by `isort`).
- No wildcard imports (`from x import *`) outside `__init__.py`.
- Circular imports are a CRITICAL issue — restructure with dependency injection
  or lazy imports.
- Type-only imports must use `TYPE_CHECKING` guard to avoid runtime cost.

---

## Dataclasses and Data Models

- Prefer `@dataclass` over plain dicts for structured internal data.
- Mutable default values must use `field(default_factory=...)`.
- `__repr__` must not expose sensitive data (private keys, tokens, passwords).
- `__eq__` on mutable classes should be used carefully — it affects dict/set usage.

---

## API Design

- Public API functions must have docstrings (Google style or ReST style —
  be consistent with the project).
- Private helpers must be prefixed with `_`.
- `__all__` must be defined in `__init__.py` for any public package.
- Deprecated functions must use `warnings.warn(..., DeprecationWarning, stacklevel=2)`.
- Breaking signature changes must have a compatibility shim with a deprecation warning
  unless the project version bump allows it.

---

## Logging

- No `print()` in library code — use the `logging` module.
- Log calls must use lazy `%s` formatting, not f-strings:
  ```python
  # CORRECT
  log.debug("connected to peer %s", peer_id)

  # WRONG — allocates string even if log level is disabled
  log.debug(f"connected to peer {peer_id}")
  ```
- Do not log private keys, passwords, or sensitive tokens at any log level.
- Use appropriate levels: `DEBUG` for tracing, `INFO` for lifecycle events,
  `WARNING` for recoverable problems, `ERROR` for failures.

---

## General

- No mutable global state (module-level mutable variables shared across calls).
- Magic numbers must be named constants with a comment explaining the value.
- Long functions (> ~50 lines) should be reviewed for single-responsibility violations.
- Dead code (commented-out blocks, unreachable branches) should be removed, not committed.
