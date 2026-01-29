# AGENTS.md — Engineering Standards

> Rules for AI agents. Be literal. Follow numbered steps. When ambiguous, ask.

---

## Core Principles

1. **Clarity over cleverness** — Write code a tired engineer can debug at 2am
2. **Explicit over implicit** — Don't hide behavior in magic
3. **Composition over inheritance** — Small, focused units that combine
4. **Fail fast, fail loud** — Surface errors at the source, not downstream
5. **Delete code** — Less code = fewer bugs. Question every addition.

---

## Code Organization

### File Structure

- One type per file
- Filename matches type name exactly
- Group by feature/domain, not by technical layer
- Shared utilities go in `Core/` or `Common/` with zero domain dependencies

### Dependency Direction

```
Features → Services → Core
    ↓          ↓        ↓
   UI      Business   Utilities
          Logic
```

- `Core/` depends on nothing internal
- `Services/` depends only on `Core/`
- `Features/` can depend on `Services/` and `Core/`
- Never create circular dependencies
- A feature should be deletable without breaking unrelated code

---

## Code Style

### Naming

| Type | Convention |
|------|------------|
| Types/Classes | `UpperCamelCase` |
| Functions/Variables | `lowerCamelCase` |
| Constants | `lowerCamelCase` or `SCREAMING_SNAKE_CASE` (lang convention) |
| Protocols/Interfaces | `UpperCamelCase`, noun or adjective (`Buildable`, `BuildService`) |

### Functions

- Do one thing
- Name describes what it does, not how
- Max 3-4 parameters — beyond that, use a configuration object
- Avoid boolean parameters — they obscure intent at call sites

```
// Bad: what does `true` mean?
build(scheme, true, false)

// Good: intent is clear
build(scheme, configuration: .release, clean: true)
```

### Comments

- Explain WHY, not WHAT
- Delete comments that restate the code
- TODO format: `// TODO: [context] description`
- Document non-obvious behavior and workarounds with context

---

## Error Handling

### Rules

1. Define domain-specific error types per module
2. Include context in errors (what failed, with what inputs)
3. Map external errors to domain errors at boundaries — don't leak implementation details
4. Fail at the source — don't pass invalid state downstream hoping someone else handles it
5. Errors are part of your API — design them as carefully as success paths

### Error Design Checklist

- [ ] Does the error message help diagnose the problem?
- [ ] Does it include relevant context (IDs, paths, values)?
- [ ] Can the caller distinguish between error types programmatically?
- [ ] Are transient vs permanent failures distinguishable?

---

## Testing

### Principles

- Tests are isolated: no shared state, no execution order dependencies
- One behavior per test
- Tests run in parallel
- Test behavior, not implementation

### Naming

```
test_{unit}_{scenario}_{expectedOutcome}
```

Examples:
- `test_build_whenSchemeNotFound_throwsError`
- `test_parse_withMalformedJSON_returnsNil`
- `test_cache_afterExpiration_fetchesNewData`

### Structure: Arrange-Act-Assert

```
func test_example() {
    // Arrange — set up preconditions
    
    // Act — execute the behavior under test
    
    // Assert — verify outcomes
}
```

One blank line between sections. No other code structure.

### Before Writing a Test

1. Verify the behavior isn't already tested
2. Identify required mock in `Tests/Mocks/` — create only if missing
3. Identify required fixture in `Tests/Fixtures/` — create only if missing

### Test Priority (when adding features)

1. Happy path for core flow
2. Error cases for core flow
3. Edge cases (only after core is solid)

### What NOT to Test

- Private implementation details (test via public interface)
- Framework/language behavior
- Trivial data containers with no logic
- Third-party library internals

---

## Mocks

**Location:** `Tests/Mocks/`  
**Naming:** `Mock{InterfaceName}`

### Mock Requirements

1. Track all inputs received (for verification)
2. Allow stubbing return values (for control)
3. Provide `reset()` method for reuse
4. One mock per interface
5. Stateless or explicitly resettable

```
MockBuildService
├── stubbedBuildResult      // Control what it returns
├── receivedConfigurations  // Verify what it received  
├── buildCallCount          // Verify how many times called
└── reset()                 // Clear state between tests
```

---

## Fixtures

**Location:** `Tests/Fixtures/`

### Rules

- Fixtures are static, deterministic test data
- Name fixtures descriptively: `validUser`, `expiredToken`, `malformedResponse`
- Keep fixtures minimal — only include fields relevant to tests
- JSON/data files go in `Fixtures/` folder
- Code-based fixtures go in `{Type}+Fixtures` extension files

---

## Dependencies

### Before Adding a Dependency

1. Can we solve this in <100 lines ourselves?
2. Is it actively maintained?
3. What's the transitive dependency cost?
4. What's the license?
5. What happens if this disappears tomorrow?

### Rules

- Wrap third-party APIs behind interfaces you control
- Pin versions explicitly
- Isolate dependency imports to wrapper modules when possible
- Update dependencies deliberately, not automatically

---

## Security

- Never log secrets, tokens, or PII (redact by default)
- Validate and sanitize all inputs at boundaries
- Prefer least-privilege configs and credentials
- Use allowlists over denylists for sensitive operations
- Store secrets in managed secret stores, never in source control
- Security-related failures should be explicit, consistent, and audited

---

## API Design & Versioning

- Version any public API and document breaking changes
- Deprecate with a clear policy and timeline
- Use semantic versioning when possible
- Backward compatibility is a feature; treat it as such

---

## Observability

- Use structured logs (not ad-hoc strings)
- Attach correlation/request IDs to key flows
- Emit metrics for core paths and failures
- Errors include stable error codes for support/debugging

---

## Performance Budgets

- Define budgets for critical paths (startup, latency, memory)
- Add guardrails to prevent regressions
- Treat performance as a product feature

---

## Documentation

- Every module/feature has a README: purpose, public API, quick usage
- Document invariants and expected lifecycle behavior
- Keep docs close to code and update with changes

---

## Configuration & Environment

- No hidden global state; config is explicit and validated at startup
- Environments must be reproducible from source control
- Fail fast on missing or invalid config

---

## Concurrency & Threading

- Document thread-safety and async expectations for public APIs
- Avoid shared mutable state; prefer immutability and message passing
- Concurrency bugs are correctness bugs—treat them as such

---

## Data Migrations

- Migrations are idempotent and forward-compatible
- Prefer reversible migrations when feasible
- Validate pre- and post-conditions

---

## Code Review

### Minimum Checklist

- [ ] Tests added or updated
- [ ] Error handling reviewed for context and clarity
- [ ] Logging/metrics added where needed
- [ ] Security and privacy considerations covered
- [ ] Performance implications considered

---

## Git Hygiene

### Commits

- One logical change per commit
- Present tense, imperative mood: "Add build caching" not "Added build caching"
- First line ≤50 chars, then blank line, then details if needed

### Branches

- `main` is always deployable
- Feature branches: `feature/{short-description}`
- Fix branches: `fix/{issue-or-description}`
- Delete branches after merge

---

## When Uncertain

1. **Check existing patterns** — How does the codebase already solve similar problems?
2. **Ask** — Ambiguity is expensive. Clarify before implementing.
3. **Smallest change** — Prefer the minimal diff that solves the problem.
4. **Reversibility** — Prefer changes that are easy to undo.
