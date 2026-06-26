# py-libp2p Repository Profile

Applied when the repository name contains `py-libp2p`.

---

## Newsfragments — BLOCKER

Newsfragments are **mandatory** for PR approval. Missing or invalid fragments
are a CRITICAL blocker — the PR cannot be merged without them.

**Directory:** `newsfragments/`
**Format:** `<ISSUE_NUMBER>.<TYPE>.rst`

Rules:
- One file per linked issue. If the PR fixes #930 and #931, both
  `930.bugfix.rst` and `931.bugfix.rst` must be present.
- If no issue exists, use the PR number.
- Valid types: `breaking`, `bugfix`, `deprecation`, `docs`, `feature`,
  `internal`, `misc`, `performance`, `removal`
- Content: user-facing ReST description. NOT developer-facing.
  Example: `"Added support for Ed25519 key generation in libp2p peer identity creation."`
- File must end with a newline character (`\n`).

Report missing/malformed fragments as:
```
Severity: CRITICAL — BLOCKER
Action required: PR cannot be approved until newsfragment is added.
```

---

## Build and Linting (MODE == full only)

After checking out the PR branch, run:

```bash
source venv/bin/activate
make pr
```

Report any errors as CRITICAL.

If documentation-related files changed (`.rst`, `.md`, `docs/`), also run:

```bash
make linux-docs
```

Report any documentation build errors as MAJOR.

---

## Architecture Layer Boundaries

Verify changes do not violate these layer boundaries.
Lower layers must NOT import from upper layers.

```
Application
    │
PubSub / Bitswap / Identify / DHT / mDNS
    │
Muxer (mplex / yamux)
    │
Security (noise / TLS)
    │
Transport (TCP / QUIC / WebSocket)
    │
Network / Swarm
    │
Peerstore / PeerID
```

Any upward dependency (e.g., Transport importing from PubSub) is a CRITICAL architecture violation.

---

## Transport Lifecycle

For any transport-related changes, verify:

- Listener starts and stops cleanly with no leaked goroutines or tasks.
- `dial()` does not leak connections on failure — ensure cleanup in the error path.
- Connection `close()` propagates to all open streams.
- `asyncio.CancelledError` is never silently swallowed.
- Timeouts use `asyncio.wait_for`, not polling loops.
- Transports implement the full `ITransport` interface.

---

## Stream Lifecycle

- Streams are explicitly closed after use (prefer `async with` or `finally` blocks).
- Half-close (EOF) is handled correctly — reading from a half-closed stream
  should return EOF, not hang.
- Writing to a closed stream raises an appropriate error.
- Stream IDs do not collide under concurrent use.

---

## Protocol Negotiation (multistream-select)

- Protocol strings follow the `/name/semver` format (e.g., `/libp2p/id/1.0.0`).
- Protocol handlers are registered before dial/listen begins.
- Unknown protocols return a proper protocol error, not a crash.

---

## Peer Identity

- PeerID is validated before insertion into the peerstore.
- No code path allows storing an unverified peer.
- Empty PeerID is rejected explicitly.
- Supported key types: Ed25519 (preferred), RSA, Secp256k1.
- Public key is not logged at INFO level or above.

---

## DHT

- Key format follows the Kademlia spec (`/pk/<peerID>`, `/ipns/<peerID>`, etc.).
- Routing table updates are protected against concurrent modification.
- Provider records have an expiry — no permanent provider records without justification.
- `find_peer` and `find_providers` have timeouts.

---

## PubSub / GossipSub

- Topic validation is applied before message forwarding.
- Message deduplication cache is bounded (has a max size or TTL).
- Message signing is enforced unless explicitly disabled with justification.
- Score parameters are not set to values that would accept all peers unconditionally.
- Fan-out maps are cleared on topic unsubscription.

---

## Bitswap

- Want-list is bounded — no unbounded growth under adversarial peers.
- Sessions are cleaned up on peer disconnect.
- Block requests have timeouts.
- Received blocks are validated before adding to the blockstore.

---

## Identify

- Protocol version and agent version strings are non-empty.
- Listen addresses sent to remote peer are filtered (no unroutable addresses
  sent to public peers unless explicitly configured).
- Observed address from remote is not trusted blindly — it is stored as
  candidate, not as confirmed listen address.

---

## mDNS

- Service name follows the `_p2p._udp.local` spec.
- Peer announcement does not advertise private/loopback addresses to peers
  that are not on the same local network.
- mDNS responder stops cleanly on `close()`.

---

## Peerstore

- All records are inserted with a TTL. Permanent records (`AddrsAddrsTTL` with
  `PermanentAddrTTL`) must be justified in the PR.
- Peer metadata values are bounded in size.
- Peerstore does not store sensitive key material in the addresses field.

---

## Async Rules (py-libp2p specific)

- `asyncio.ensure_future` is discouraged; prefer explicit task tracking
  (`self._tasks: list[asyncio.Task]`).
- All background tasks must be cancelled and awaited in `close()`.
- `CancelledError` must never be caught and discarded — always re-raise.
- All `async def` functions that are called must be awaited.
- `asyncio.shield` usage must include a comment explaining why it is needed.
- `asyncio.get_event_loop()` is deprecated in Python 3.10+ — use
  `asyncio.get_running_loop()`.

---

## Interface Compliance

Verify that abstract base class interfaces are still fully satisfied:
`INetwork`, `ITransport`, `IListener`, `IStream`, `IMuxedConn`,
`ISecureConn`, `IPeerStore`.

Any removed or renamed method on a public interface is a **breaking change**
and requires a `.breaking.rst` newsfragment.

---

## Interoperability

For transport or protocol-level changes, note whether go-libp2p or
js-libp2p interoperability tests exist and whether they need to be
updated or added.
