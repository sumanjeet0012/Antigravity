# PR Review Skill

A reusable, repository-aware AI skill for reviewing pull requests.
Works for any repository. Loads a specialized profile for known projects.

---

## Quick Start

```
Review PR #1029
Quick review of PR #1029
Security review of PR #1029
Architecture review of PR #1029
Performance review of PR #1029
```

Reports are saved to:
```
downloads/AI-PR-REVIEWS/<PR_NUMBER>-AI-PR-REVIEW.md
```

---

## Review Modes

| Mode | Focus | When to Use |
|---|---|---|
| `full` (default) | Everything | Standard reviews |
| `quick` | Critical + Major only | Time-sensitive, draft PRs |
| `security` | Auth, crypto, input validation | Security-sensitive changes |
| `architecture` | Layering, cohesion, coupling | Structural / design PRs |
| `performance` | Allocations, IO, async | Performance-sensitive paths |

---

## Supported Repository Profiles

| Repository | Profile File | Extra Checks |
|---|---|---|
| `py-libp2p` | `profiles/py-libp2p.md` | Newsfragments (BLOCKER), transport lifecycle, stream lifecycle, DHT, PubSub, Bitswap, Identify, mDNS, peerstore, interface compliance, make pr, make linux-docs |
| `unified-testing` | `profiles/unified-testing.md` | YAML validity (conflict markers), CI workflow correctness, implementation registry consistency |
| Any other repo | `profiles/generic.md` | Newsfragments (if towncrier used), changelog, CI |

---

## Supported Language Rules

| Language | Rules Files |
|---|---|
| Python | `rules/python.md` + `rules/async-python.md` |
| Go | `rules/go.md` |
| JavaScript / TypeScript | `rules/javascript.md` |
| Rust | Generic correctness rules (no Rust file yet) |

---

## File Structure

```
pr-review-skill/
├── SKILL.md                    ← Load this. Master orchestrator.
├── README.md                   ← This file.
│
├── profiles/
│   ├── generic.md              ← Fallback for any unknown repo
│   ├── py-libp2p.md            ← py-libp2p specific rules
│   └── unified-testing.md      ← unified-testing specific rules
│
├── rules/
│   ├── python.md               ← Python type hints, exceptions, API design
│   ├── async-python.md         ← asyncio / trio: awaits, cancellation, tasks
│   ├── go.md                   ← Go error handling, goroutines, context
│   └── javascript.md           ← JS/TS promises, types, security
│
├── modes/
│   ├── full.md                 ← Comprehensive (default)
│   ├── quick.md                ← Fast, blockers only
│   ├── security.md             ← Security-focused
│   ├── architecture.md         ← Design-focused
│   └── performance.md          ← Efficiency-focused
│
└── templates/
    └── report.md               ← 14-section structured report template
```

---

## MCP vs CLI

The skill prefers **GitHub MCP** for collecting PR context.
It falls back to **GitHub CLI** (`gh`) automatically if MCP is unavailable.

No configuration required — the skill detects what is available.

---

## Adding a New Repository Profile

1. Create `profiles/<repo-name>.md` following the structure of an existing profile.
2. Add a detection row to the **Load repository profile** table in `SKILL.md` Phase 3.
3. Add a row to the table in this README.

---

## Adding a New Language

1. Create `rules/<language>.md`.
2. Add a detection row to the **Load language rules** table in `SKILL.md` Phase 3.
3. Add a row to the table in this README.

---

## Adding a New Review Mode

1. Create `modes/<mode-name>.md`.
2. Add the mode trigger to the **Trigger Phrases** table in `SKILL.md`.
3. Add the mode to the **Load review mode** table in `SKILL.md` Phase 3.
4. Add a row to the modes table in this README.

---

## Roadmap

Future repository profiles planned:
- `GooseSwarm`
- `py-ipfs-lite`
- `universal-connectivity`
- `antigravity`

Future skills that will reuse this workflow:
- `issue-investigation`
- `github-actions-debug`
- `protocol-design`
- `security-audit`
