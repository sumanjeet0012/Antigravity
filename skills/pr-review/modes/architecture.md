# Architecture Review Mode

Focused on design quality, cohesion, long-term maintainability, and extensibility.

---

## Focus Areas

### Layering and Boundaries
- Changes respect the established architectural layers of the project.
- No upward dependencies (lower-level modules must not import from
  higher-level modules).
- No new circular dependencies between modules or packages.
- If a layer boundary is crossed, the PR description must justify it.

### Single Responsibility
- Each module, class, and function has one clear, named responsibility.
- No "god objects" (classes with > ~5 distinct responsibilities).
- Functions longer than ~50 lines are reviewed for decomposition opportunities.

### Coupling and Cohesion
- New code depends on abstractions (interfaces, protocols), not concrete
  implementations.
- Dependencies are injected (constructor or function parameter), not hardcoded.
- No new global mutable state introduced.
- Cohesion: methods in a class use the class's own fields — unrelated methods
  are a sign of poor cohesion.

### Extensibility
- New functionality uses existing extension points where possible.
- New extension points (hooks, plugins, interfaces) are documented.
- Hardcoded behavior that the caller might need to override is a design smell.

### Backward Compatibility
- Public API additions: additive changes are safe.
- Public API removals or renames: CRITICAL — must have a migration path,
  deprecation warning, and `.breaking.rst` newsfragment.
- Behavioral changes to existing APIs: document in the PR and changelog.

### Abstractions
- Abstractions introduced here — are they pulling their weight?
  (Over-abstraction is as bad as under-abstraction.)
- Interface methods are minimal — no "fat" interfaces that force implementers
  to provide methods they do not use.

---

## What to Report

Focus heavily on Architecture section.
Include all Critical and Major correctness issues.
Omit performance notes and minor style issues unless they directly affect
architecture quality.
