# Async Python Review Rules

Applied to all Python repositories using `asyncio` or `trio`.

---

## CRITICAL Issues

### Unawaited Coroutines

Every `async def` call must be `await`ed or explicitly wrapped in a `Task`.
An unawaited coroutine silently does nothing.

```python
# BAD — coroutine created but never executed
result = some_async_function()

# GOOD
result = await some_async_function()
```

Flag any call to an `async def` function without `await` as CRITICAL.

---

### CancelledError Swallowing

`asyncio.CancelledError` (Python 3.8+) inherits from `BaseException`.
Catching it without re-raising breaks cooperative cancellation.

```python
# BAD — drops the cancellation signal, task never stops
try:
    await something()
except asyncio.CancelledError:
    pass

# GOOD — always re-raise after cleanup
try:
    await something()
except asyncio.CancelledError:
    await cleanup()
    raise
```

Any `except asyncio.CancelledError` or `except (Exception, CancelledError)` that
does not re-raise is a CRITICAL issue.

---

### Task Leaks

Background tasks that are not tracked and cancelled on shutdown will run
after the owning object is closed, causing use-after-close bugs and
preventing clean shutdown.

```python
# BAD — fire and forget
asyncio.ensure_future(background_work())

# GOOD — tracked and cancelled on close
self._tasks: list[asyncio.Task] = []

task = asyncio.ensure_future(background_work())
self._tasks.append(task)

async def close(self) -> None:
    for task in self._tasks:
        task.cancel()
    await asyncio.gather(*self._tasks, return_exceptions=True)
    self._tasks.clear()
```

---

### Blocking IO Inside Async Function

Synchronous IO (file reads, DNS lookups, subprocess.run) called directly inside
an `async def` blocks the entire event loop.

```python
# BAD — blocks the event loop
async def read_config(path: str) -> str:
    return open(path).read()

# GOOD — offload to thread pool
async def read_config(path: str) -> str:
    loop = asyncio.get_running_loop()
    return await loop.run_in_executor(None, Path(path).read_text)
```

Flag any `open()`, `socket.connect()`, `subprocess.run()`, `time.sleep()`,
or equivalent blocking calls inside `async def` as CRITICAL.

---

### Missing Timeouts on Network Operations

Network operations without a timeout will hang indefinitely if the remote
peer is unresponsive.

```python
# BAD — hangs forever
data = await stream.read(1024)

# GOOD — raises asyncio.TimeoutError after 30 seconds
data = await asyncio.wait_for(stream.read(1024), timeout=30.0)
```

---

## MAJOR Issues

### Lock Misuse

- `asyncio.Lock` must be used with `async with`, not `lock.acquire()` / `lock.release()`.
  Manual acquire/release leaks the lock if an exception occurs between them.
- Holding a lock across multiple `await` points when the lock is not needed for
  all of them is a concurrency bottleneck.
- A single global lock serializing unrelated operations is a performance issue.

---

### asyncio.shield Misuse

`asyncio.shield` protects the inner coroutine from the *current* cancellation
but not from all cancellations. It is often misused to "prevent cancellation"
when it does not actually do so.

Any use of `asyncio.shield` must include a comment explaining:
1. What it is protecting.
2. Why the protected operation must complete even if the caller is cancelled.
3. How the protected task is eventually cleaned up.

---

### Event Loop Access

- `asyncio.get_event_loop()` is deprecated in Python 3.10+ inside coroutines.
  Use `asyncio.get_running_loop()` inside `async def`, or `asyncio.new_event_loop()`
  at the top level.
- Never call `loop.run_until_complete()` inside a coroutine (it will raise).
- Never create a new event loop inside an `async def` function.

---

### Async Generator Cleanup

`async for` over an async generator that raises mid-iteration must call
`aclose()` on the generator to trigger its `finally` block.
Use `async with contextlib.aclosing(gen)` to guarantee cleanup.

---

## MINOR Issues

- `await asyncio.sleep(0)` as a voluntary yield point is acceptable but
  should have a comment explaining why it is needed.
- `asyncio.gather(*tasks)` without `return_exceptions=True` will raise on
  the first failure and leave remaining tasks running — use `return_exceptions=True`
  when all results are needed.
- Nested `asyncio.wait_for` calls with different timeouts — verify the inner
  timeout is shorter, otherwise the outer timeout is effectively ignored.
