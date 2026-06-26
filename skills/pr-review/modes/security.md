# Security Review Mode

Focused security-first review. Use for PRs touching auth, crypto, transports, RPC, or serialization.

---

## Focus Areas

### Input Validation
- All data from external peers, network sockets, or user input is validated
  before use.
- No format-string injection vulnerabilities.
- No path traversal via user-supplied file paths (`../` sequences).
- Integer overflow or underflow on size/length parameters.
- Protobuf / msgpack / JSON fields are bounds-checked before allocation.

### Authentication and Peer Identity
- PeerID verification is not bypassed or skipped.
- Cryptographic signatures are verified before trusting signed data.
- No hardcoded credentials, API keys, or secrets in source code or config files.
- Private key material is not logged or included in error messages.

### Cryptography
- No custom cryptographic implementations — use well-audited libraries.
- Algorithm selection: no MD5 or SHA-1 for security-sensitive operations.
- Random number generation uses `secrets` (Python), `crypto/rand` (Go),
  or OS-provided CSPRNG — not `random` or `Math.random()`.
- Key lengths meet current standards: RSA ≥ 2048 bits, EC ≥ 256 bits.

### Network Attack Surface
- No SSRF: user-controlled URLs or addresses are not used in internal requests
  without validation against an allowlist.
- Peer-supplied routing information is not trusted unconditionally.
- No amplification attack vectors (small request → large response to third party).
- Rate limiting is present on public-facing endpoints or high-cost operations.

### Secrets and Sensitive Data
- No secrets in log output (any level).
- No secrets visible in stack traces or exception messages.
- No secrets accidentally introduced in the diff (check for API keys,
  private keys, password strings).
- Environment variable handling does not log env values.

### Serialization and Deserialization
- Untrusted data is never deserialized with `pickle`, `marshal`, `yaml.load`
  (unsafe loader), or equivalent.
- Schema validation is applied before processing deserialized data.
- Recursive data structures from untrusted input have depth limits.

---

## What to Report

Focus heavily on the Security Review section (section 7).
Still include all Critical correctness issues.
Omit style, documentation, and minor notes.
