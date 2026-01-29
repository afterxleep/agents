# AGENTS.md — Engineering Standards

> Rules for AI agents. Be literal. Follow numbered steps. When ambiguous, ask.

---

## Core Principles

1. **Clarity over cleverness** — Write code a tired engineer debugs at 2am
2. **Explicit over implicit** — No magic, no surprises
3. **Composition over inheritance** — Small units that combine
4. **Fail fast, fail loud** — Errors surface at the source
5. **Delete code** — Less code = fewer bugs. Question every addition.
6. **Verify, don't assume** — Run it. Test it. Prove it works.

---

## Feature Development Workflow

### Before Writing Code

1. **Clarify requirements** — Restate the goal. Ask if ambiguous.
2. **Identify failure modes** — List what can go wrong:
   - Invalid inputs
   - Missing dependencies
   - Network/IO failures
   - Concurrency issues
   - Resource exhaustion
3. **Classify by priority**:
   - **Core flow**: Happy path + direct error cases
   - **Edge cases**: Unusual but valid scenarios
   - **Out of scope**: Document, don't implement
4. **Check existing code** — Is this already solved? Can you extend rather than create?

### Implementation Order

1. Write failing test for core happy path
2. Implement minimum code to pass
3. Write failing tests for core error cases
4. Implement error handling
5. Refactor if needed (tests stay green)
6. Add edge case tests only after core is solid

### Before Submitting

- [ ] All tests pass
- [ ] No commented-out code
- [ ] No TODO without context (`// TODO: [reason] description`)
- [ ] Error messages are actionable
- [ ] No secrets, credentials, or hardcoded environment-specific values
- [ ] Formatting/linting passes

---

## Debugging Process

### When Something Fails

1. **Reproduce reliably** — Can you trigger it consistently?
2. **Isolate the scope** — What's the smallest input that fails?
3. **Read the error** — Actually read it. Full stack trace.
4. **Form a hypothesis** — One specific guess about the cause
5. **Test the hypothesis** — Add logging, write a test, or inspect state
6. **Fix and verify** — Change one thing. Confirm it's fixed.
7. **Add regression test** — Ensure it can't silently break again

### Don't

- Change multiple things at once
- Assume you know the cause without evidence
- Delete error handling to "simplify"
- Fix symptoms instead of root causes

---

## Code Organization

### File Structure

- One type per file
- Filename matches type name exactly
- Group by feature/domain, not by layer
- Shared utilities in `Core/` or `Common/` — zero domain dependencies

### Dependency Direction

```
Features → Services → Core
    ↓          ↓        ↓
   UI      Business   Utilities
           Logic
```

- `Core/` depends on nothing internal
- `Services/` depends only on `Core/`
- `Features/` depends on `Services/` and `Core/`
- No circular dependencies
- Features are deletable without breaking unrelated code

---

## Code Style

### Naming

| Type | Convention |
|------|------------|
| Types/Classes | `UpperCamelCase` |
| Functions/Variables | `lowerCamelCase` |
| Constants | `lowerCamelCase` or `SCREAMING_SNAKE_CASE` (follow language convention) |
| Protocols/Interfaces | `UpperCamelCase`, noun or adjective |

### Functions

- Do one thing
- Name describes what, not how
- Max 3-4 parameters — beyond that, use a config object
- Avoid boolean parameters — they obscure intent at call sites

```
// Bad
build(scheme, true, false)

// Good
build(scheme, configuration: .release, clean: true)
```

### Comments

- Explain WHY, not WHAT
- Delete comments that restate code
- TODO format: `// TODO: [context] description`
- Document non-obvious behavior and workarounds

---

## Error Handling

### Rules

1. Define domain-specific error types per module
2. Include context: what failed, with what inputs
3. Map external errors at boundaries — don't leak implementation details
4. Fail at the source — don't pass invalid state hoping someone handles it
5. Errors are API — design them like success paths

### Error Checklist

- [ ] Message helps diagnose the problem
- [ ] Includes relevant context (IDs, paths, values)
- [ ] Caller can distinguish error types programmatically
- [ ] Transient vs permanent failures are distinguishable

---

## Testing

### Principles

- Tests are isolated: no shared state, no execution order dependencies
- One behavior per test, descriptive names: `test_build_failsWhenSchemeNotFound`
- Tests run in parallel — design for it
- Test behavior, not implementation
- Fast tests get run; slow tests get skipped

### Before Writing a Test

1. Check if the **behavior** is already tested (not just the code path — multiple tests can hit the same lines testing different behaviors)
2. Identify the mock in `Tests/Mocks/` — create only if missing
3. Identify the fixture in `Tests/Fixtures/` — create only if missing

### When Adding Features

1. **Document failure modes first** — List edge cases and failure scenarios in a comment block before writing implementation code:
   ```
   // Failure modes:
   // - Scheme not found in project
   // - Build timeout exceeded  
   // - Simulator unavailable
   // - Concurrent build in progress
   ```
2. Write tests in this order:
   - Happy path for core flow
   - Error/failure cases for core flow
   - Edge cases (only after core flow is solid)
3. Don't skip ahead — core flow must be solid before edge cases

### Test Naming

```
test_{unit}_{condition}_{expectedResult}
```

Examples:
- `test_build_failsWhenSchemeNotFound`
- `test_parse_returnsNilForMalformedJSON`
- `test_cache_fetchesNewDataAfterExpiration`

### Structure: Arrange-Act-Assert

```
func test_example() {
    // Arrange — setup preconditions
    
    // Act — execute behavior under test
    
    // Assert — verify outcomes
}
```

One blank line between sections. No other structure.

### What to Test

| Test | Don't Test |
|------|------------|
| Public interface behavior | Private implementation details |
| Error handling paths | Framework/language behavior |
| State transitions | Trivial getters/setters |
| Business logic | Third-party library internals |

### Unit vs Integration

- **Unit**: Single component, mocked dependencies, fast
- **Integration**: Multiple components, real dependencies, slower

Default to unit tests. Use integration tests for:
- Critical paths that must work end-to-end
- Complex component interactions
- External service contract validation

---

## Mocks

**Location:** `Tests/Mocks/`  
**Naming:** `Mock{ProtocolName}` (e.g., `MockBuildService`)

### Rules

1. One mock per protocol/interface
2. Mocks must be stateless or resettable between tests
3. Track all inputs received (for verification)
4. Allow stubbing return values (for control)
5. Provide `reset()` method to clear state

### Structure

```
MockBuildService
├── stubbedResult           // Control what it returns
├── receivedConfigurations  // Verify what it received
├── callCount               // Verify how many times called
└── reset()                 // Clear state between tests
```

### Don't

- Create test-specific mocks — reuse shared mocks
- Let mocks accumulate state across tests
- Mock what you don't own (wrap it first, mock the wrapper)

---

## Fixtures

**Location:** `Tests/Fixtures/`

- Static, deterministic test data
- Named descriptively: `validUser`, `expiredToken`, `malformedResponse`
- Minimal — only fields relevant to tests
- JSON/data files in `Fixtures/` folder
- Code-based fixtures in `{Type}+Fixtures` extension files

---

## Refactoring

### When to Refactor

- Before adding a feature (make the change easy, then make the easy change)
- After tests pass (not during implementation)
- When you touch code that's hard to understand

### When NOT to Refactor

- While debugging
- Without test coverage
- Unrelated to the current task
- "While I'm here" changes — make a separate commit or ticket

### How to Refactor

1. Ensure tests exist and pass
2. Make one structural change
3. Run tests
4. Repeat

Never change behavior and structure in the same step.

---

## Dependencies

### Before Adding

1. Can we solve this in <100 lines ourselves?
2. Is it actively maintained?
3. What's the transitive dependency cost?
4. What's the license?
5. What if it disappears tomorrow?

### Rules

- Wrap third-party APIs behind interfaces you control
- Pin versions explicitly
- Isolate imports to wrapper modules
- Update deliberately, not automatically

---

## Git Hygiene

### Commits

- One logical change per commit
- Present tense, imperative: "Add caching" not "Added caching"
- First line ≤50 chars, blank line, then details
- Follow GIT templates if present in the repository

### Branches

- `main` is always deployable
- Feature: `feature/{description}`
- Fix: `fix/{description}`
- Delete after merge

---

## When Uncertain

1. **Check existing patterns** — How does the codebase solve similar problems?
2. **Ask** — Ambiguity is expensive. Clarify before implementing.
3. **Smallest change** — Prefer minimal diff that solves the problem.
4. **Reversibility** — Prefer changes easy to undo.
5. **Prove it** — Run the code. Pass the tests. Don't guess.
