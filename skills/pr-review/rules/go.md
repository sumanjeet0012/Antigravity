# Go Review Rules

Applied to Go repositories.

---

## Error Handling

- Every error return must be checked. Discarding with `_` requires a comment.
- Error wrapping must use `fmt.Errorf("context: %w", err)` to preserve the chain.
- Use `errors.Is` and `errors.As` for comparison — not `==` on error values.
- Sentinel errors (`var ErrX = errors.New(...)`) must be exported only if callers
  need to match against them.
- `log.Fatal`, `os.Exit`, and `panic` must not appear in library code —
  only in `main()` or test helpers.

---

## Goroutines

- Every goroutine must have a clear owner responsible for its lifecycle.
- `context.Context` must be the first parameter of any function that starts IO
  or may block.
- Use `sync.WaitGroup` when the caller needs to wait for goroutine completion.
- Channel sends must not block indefinitely — use `select` with `ctx.Done()`:
  ```go
  select {
  case ch <- value:
  case <-ctx.Done():
      return ctx.Err()
  }
  ```
- Goroutine leaks (goroutines that outlive their expected scope) are a CRITICAL issue.

---

## Interfaces

- Interface definitions should be small (1–3 methods is ideal).
- Interfaces are satisfied implicitly — verify the intended concrete type
  satisfies the interface by adding a compile-time check:
  ```go
  var _ MyInterface = (*MyImpl)(nil)
  ```
- `interface{}` / `any` usage must be justified — prefer typed alternatives.

---

## Context

- First parameter of public functions doing IO or blocking must be `context.Context`.
- Never store a `context.Context` in a struct field — pass it as a parameter.
- `context.Background()` is only appropriate at top-level entry points
  (main, test setup, long-lived server loops).
- `context.WithTimeout` and `context.WithDeadline` must always have their
  cancel function deferred:
  ```go
  ctx, cancel := context.WithTimeout(parent, 30*time.Second)
  defer cancel()
  ```

---

## Concurrency

- `sync.Mutex` fields must not be copied — embed by pointer or ensure the
  struct is never copied after first use.
- Race conditions: check for concurrent map read/write (use `sync.Map` or
  a mutex-protected map).
- `sync.Once` is the correct pattern for lazy initialization.

---

## Testing

- Table-driven tests are preferred for multiple input/output cases.
- `t.Parallel()` should be used for independent subtests.
- Never use `time.Sleep` in tests — use channels, `Eventually`, or
  `testify/require`.
- Race detector must pass: `go test -race ./...`
- Benchmark functions must use `b.ReportAllocs()` if allocation count matters.

---

## Linting

Verify the following pass without errors:
- `go vet ./...`
- `golangci-lint run` (if configured in the repo)
- `go build ./...`
- `go test ./...`
