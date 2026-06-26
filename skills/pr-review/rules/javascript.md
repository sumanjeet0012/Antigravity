# JavaScript / TypeScript Review Rules

Applied to JavaScript and TypeScript repositories.

---

## TypeScript Strictness

- `any` type requires a comment justifying why it cannot be narrowed.
- `as` type assertions must include a comment — they suppress type errors
  without runtime validation.
- `strictNullChecks: true` must be enabled in `tsconfig.json`.
- Return types must be annotated on all exported functions.
- `unknown` is preferred over `any` for values from external sources
  (JSON parse, API responses).

---

## Async and Promises

- Every `Promise` must be `await`ed or have a `.catch()` handler.
  Unhandled rejections are a CRITICAL issue.
- Never wrap a Promise-returning function in `new Promise()` — this is
  the "explicit Promise construction antipattern":
  ```js
  // BAD
  return new Promise((resolve) => {
    fetchData().then(resolve);
  });

  // GOOD
  return fetchData();
  ```
- `async` callback inside `Array.forEach` does not await — the forEach
  returns before the callbacks complete. Use `for...of` or `Promise.all`:
  ```js
  // BAD — forEach does not wait for async callbacks
  items.forEach(async (item) => { await process(item); });

  // GOOD
  for (const item of items) { await process(item); }
  // OR
  await Promise.all(items.map(async (item) => process(item)));
  ```
- Mixing `Promise.all` with individual `await` calls when results are
  independent is a performance issue — use `Promise.all` for parallel IO.

---

## Error Handling

- `try/catch` blocks must not silently swallow errors.
  Always log, rethrow, or return the error.
- In TypeScript, `catch (err)` catches `unknown` — always narrow before use:
  ```ts
  catch (err: unknown) {
    if (err instanceof Error) {
      console.error(err.message);
    }
  }
  ```
- `JSON.parse` must always be wrapped in `try/catch` — it throws on invalid input.

---

## Module System

- Consistent use of ESM (`import/export`) or CJS (`require/module.exports`)
  within a package — do not mix.
- Named exports preferred over default exports in library code (they are
  more refactor-safe and easier to tree-shake).
- Circular imports are a CRITICAL issue.
- Dynamic `import()` must be justified — it defers loading and can hide
  missing module errors.

---

## Security

- No `eval()` or `new Function(string)` — these execute arbitrary code.
- No direct `innerHTML` assignment with user-supplied content — use
  `textContent` or a sanitization library.
- `JSON.parse` on untrusted input: validate the parsed shape before use
  (use a schema validator or type guard).
- No `child_process.exec(userInput)` — use `execFile` with a fixed command
  and separate argument array.

---

## Testing

- Use `async/await` in tests, not callbacks or raw `.then()` chains.
- Mock cleanup must happen in `afterEach`, not `afterAll`, to prevent
  test cross-contamination.
- Snapshot tests must be reviewed carefully — auto-updating snapshots can
  hide regressions.
- `console.error` calls in production code must be tested (assert they
  are called or silenced in tests).
