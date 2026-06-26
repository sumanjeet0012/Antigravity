# Performance Review Mode

Focused on runtime efficiency and resource usage.

---

## Focus Areas

### Memory
- No unbounded collections: lists, dicts, deques, or queues that grow
  without limit under adversarial or high-load input.
- Caches must have a bounded size (LRU, TTL, or max-entries eviction).
- Large data structures (buffers, bitmaps, byte arrays) are not held longer
  than needed — release references early.
- Copying large byte buffers unnecessarily (e.g., slicing to pass a subset)
  when a view or memoryview would suffice.

### CPU Complexity
- No O(n²) or worse algorithms on inputs that could be large in production.
- No repeated linear scans over collections that could be indexed.
- Tight inner loops: cache attribute lookups in a local variable.
- Avoid repeated string concatenation in loops — use `"".join()` or a buffer.

### IO Efficiency
- Blocking IO must never be called inside an async function (see async rules).
- No unnecessary round-trips: batch reads/writes where possible.
- Connection pooling is used where the library supports it.
- Avoid opening and closing connections per-operation when a persistent
  connection is appropriate.

### Async Efficiency
- The event loop must not be blocked by CPU-intensive operations — offload
  with `run_in_executor`.
- Use `asyncio.gather` for independent IO operations instead of sequential
  `await` calls.
- No busy-wait loops (`while True: await asyncio.sleep(0)`).
- `asyncio.Queue` with no `maxsize` is an unbounded buffer — set a limit.

### Serialization
- Large messages serialized/deserialized on the hot path — consider
  lazy parsing or streaming formats.
- Protobuf fields with `repeated` should not be re-allocated on every call.

### Benchmarks
- If the PR includes benchmark results or profiling data, reference them
  in the report.
- If a performance-sensitive path is changed without benchmarks, recommend
  adding them.

---

## What to Report

Focus heavily on the Performance Review section.
Include all Critical correctness issues.
Omit documentation suggestions and minor style issues.
