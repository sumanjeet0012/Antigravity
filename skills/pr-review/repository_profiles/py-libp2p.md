# py-libp2p Rules

Apply only for py-libp2p.

- Validate transport -> security -> muxer -> stream lifecycle.
- Check asyncio correctness (awaits, cancellation, task leaks).
- Review DHT, PubSub, Bitswap, Identify, mDNS impacts.
- Preserve interoperability with go-libp2p, rust-libp2p and js-libp2p.
- Run `make pr`.
- Run `make linux-docs` when docs/public APIs change.
- Verify required newsfragment.
- Preserve protocol compatibility and public APIs.
