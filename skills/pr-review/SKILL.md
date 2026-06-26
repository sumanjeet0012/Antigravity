# PR Review Skill

Repository-aware PR review skill.

## Workflow
1. Detect repository.
2. Use GitHub MCP to gather PR, issues, comments and CI.
3. Detect repository profile.
4. Apply generic engineering review.
5. If repository is py-libp2p, load repository_profiles/py-libp2p.md.
6. Run validation (`make pr`; `make linux-docs` only when needed).
7. Generate report from templates/review_template.md.
8. Save to downloads/AI-PR-REVIEWS/<PR>-AI-PR-REVIEW.md.

## Modes
- quick
- full
- security
- architecture

## Rules
- Prefer GitHub MCP over gh CLI.
- Never invent bugs.
- Separate facts from suggestions.
- Report confidence for uncertain findings.
