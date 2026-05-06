# Swift Engineering Guide

This guide defines how Swift should be written across a production codebase. It is intentionally practical. The goal is not to enforce fashionable patterns or idealized architecture. The goal is to produce code that is easy to understand, safe to change, straightforward to test, and resilient in production.

It is written for a real codebase with existing modules, existing operational complexity, and a need for steady improvement over time. It should be strict enough to guide refactors and rewrites later, but realistic enough to adopt immediately.

The standard is clarity first. When choosing between brevity and readability, choose readability. When choosing between cleverness and maintainability, choose maintainability. When choosing between abstraction and explicitness, choose the design that makes ownership, behavior, and failure easiest to reason about.

## Available Skills

Specialized Claude skills are installed for common Swift work. Invoke them via the `Skill` tool instead of improvising guidance from scratch. Use the right one for the task; do not stack multiple unless the work genuinely spans them.

- **`swift-concurrency`** — async/await, actors, `Sendable`, `@MainActor`, Swift 6 migration, data race fixes, refactoring closures to async/await, concurrency lint warnings. Use for any concurrency design or fix.
- **`swiftui-ui-patterns`** — building or refactoring SwiftUI views and components, `TabView` architecture, screen composition, component-specific patterns. Use when writing new UI or restructuring existing views.
- **`swiftui-view-refactor`** — cleaning up SwiftUI view files: structure, dependency injection, `@Observable` usage, non-optional view models. Use for view-level cleanup that doesn't change behavior.
- **`swiftui-performance-audit`** — diagnosing slow rendering, jank, excessive view updates, layout thrash, high CPU/memory in SwiftUI. Use when investigating runtime performance, not code style.

If a task touches one of these areas, the skill takes precedence over memorized patterns — its guidance is current and project-aware.

## 1. Design Philosophy

Write Swift for the next engineer, not for the current author. Favor directness over cleverness, explicit control flow over abstraction for its own sake, and small composable units over large multipurpose types. A good implementation should make state, ownership, failure, and side effects obvious from the call site.

The default style is explicit and boring in the best sense. Values should be typed, dependencies should be visible, errors should describe the failed operation, and concurrency should be isolated deliberately. "It probably works" is not an acceptable design standard.

Code should be easy to change safely. That means avoiding hidden coupling, reducing ambient global state, and keeping the boundaries between pure logic and side effects clear. The point of the style guide is not aesthetic consistency. The point is to make refactoring cheaper and production failures easier to reason about.

## 2. Code Organization

Organize code by domain, capability, or subsystem. Good groupings are things like build, workspace, licensing, state, automation, networking, logging, or transport. Avoid organizing primarily by technical layer such as `Models`, `Helpers`, `Managers`, and `Utils` at the top level. Those buckets usually become dumping grounds.

A file should have one primary reason to exist. In practice, that usually means one primary type per file, with small local extensions allowed when they improve readability. If a file contains unrelated helper types that could be deleted independently, the file is too broad.

Types should follow clear ownership rules. Use `struct` for values, configuration, request and response payloads, snapshots, persisted state, and parsed results. Use `enum` for finite state, options, and domain errors. Use `final class` for long-lived services, lifecycle coordinators, and types that own external resources or mutable identity. Use `actor` for shared mutable async state. Use `@MainActor` for UI-facing state.

Names should be literal. Types are nouns. Protocols describe capability. Functions describe behavior. Avoid names that describe implementation details instead of intent. A reader should be able to guess what a type owns and what a function does without opening the body.

## 3. API Design

Public and internal APIs should make misuse difficult. Prefer parameter labels that read like a sentence. Prefer domain types over primitive combinations. If a function needs many related inputs, introduce a typed input or configuration object rather than expanding the parameter list.

Boolean parameters should be treated as a smell. They hide behavior at the call site and usually indicate that the function is doing more than one thing. Replace them with enums, option structs, or separate functions when the behavior is materially different.

Keep entrypoints thin. In a CLI codebase, command types should parse flags, validate inputs, resolve the environment, delegate to a domain service, and render output. They should not embed business rules, persistence logic, or orchestration that belongs in the subsystem itself. The same principle applies to controllers and SwiftUI views.

Side-effect boundaries should be explicit. A function that touches disk, starts a process, performs a network request, mutates shared state, and formats output is too broad. Split pure transformation from operational work. If a function mixes validation, IO, and output rendering, factor it until each part has a clear contract.

Prefer return values that model outcomes directly. Use typed result objects when a workflow returns meaningful structured output. Use `Void` only when there is truly no useful outcome beyond successful completion.

## 4. Dependency Management and Abstraction

Use protocols at the edges of the system, not everywhere. Protocols are appropriate for dependencies that need to be substituted, mocked, or decoupled from infrastructure: storage, process execution, network clients, backend adapters, and external tool wrappers. They are not mandatory for every type.

Inject dependencies explicitly. Constructor injection is the default for long-lived services. Convenience defaults are acceptable for leaf production types, but core workflows should not rely on hidden singletons or implicit globals. A type's behavior should be understandable from the dependencies it receives.

Wrap external systems behind interfaces you control. Shell commands, `Process`, filesystem access, user defaults, remote APIs, and third-party libraries should be isolated behind small boundaries so the rest of the codebase depends on stable domain behavior rather than vendor details.

Abstractions must pay rent. Do not introduce a protocol, wrapper, or generic layer unless it improves testability, decoupling, or clarity. Extra layers that merely rename concrete behavior without reducing coupling should be avoided.

## 5. Error Handling

Error handling is part of API design. Each domain boundary should expose typed errors that describe the failed operation in terms meaningful to the caller. Errors should include enough context to diagnose the failure quickly: path, identifier, command, invalid value, current state, or relevant environment.

Do not leak low-level errors upward unchanged unless the boundary itself is intentionally low-level. Translate `Foundation`, decoding, filesystem, process, and network errors into domain language at module boundaries. The caller should not need to infer what failed from a generic underlying error string.

Prefer `throws` for operational failures. Use `Result` only when failure must be carried as data, such as queued responses, transport callbacks, or stored outcomes. Avoid using `Optional` to hide failure when the difference between "missing" and "failed" matters.

Use `try?` sparingly. It is only acceptable when failure is truly non-fatal and intentionally discarded. Silent failure should be rare and obvious in code review. If failure is intentionally ignored, make that intent explicit either in naming or with a brief comment.

Fail early and at the edge. Validate user input, required files, command availability, and preconditions before starting expensive work. Invalid state should not travel deeper into the system in the hope that another layer will reject it later.

## 6. State and Persistence

Persisted state is a first-class design concern. Stored models should be typed, version-aware, migration-safe, and conservative about defaults. Backward compatibility rules should be owned by the persistence layer, not spread across callers.

Normalization belongs in storage. Canonical path handling, legacy migration, schema fallback, and default decoding should live close to the stored format. Callers should consume stable domain data rather than reimplementing cleanup logic.

Persist durable product state, not transient UI or operational noise. Avoid storing values that are purely derived, short-lived, or trivially recomputed. Keep stored data minimal and intentional.

Writes should be atomic when correctness matters. Storage APIs should make it hard to partially write or corrupt important state. If locking, migration, or recovery logic exists, it should be centralized in the storage layer and tested directly.

## 7. Concurrency

Concurrency must be explicit. Shared mutable state should never rely on undocumented assumptions about thread affinity. If state is mutated from async contexts, isolate it with `actor` or `@MainActor`. If you must use locks for boundary or performance reasons, the synchronization strategy must be narrow, well-contained, and documented.

Task ownership should be clear. If a type starts async work, it should also own cancellation and cleanup. Long-lived tasks, stream consumers, timers, and observers should not float around without lifecycle control. Resource cleanup is part of correctness, not polish.

Cancellation should be propagated deliberately in long-running workflows. Process execution, streaming operations, and polling loops should support cancellation and terminate cleanly. Code that ignores cancellation tends to create the hardest operational bugs.

`Sendable` should be treated seriously. Conform only when the type's semantics are truly safe across concurrency boundaries. `@unchecked Sendable` is an escape hatch and should require a comment describing why it is safe and what synchronization guarantees exist.

Use asynchronous APIs end to end where possible. Avoid bouncing between callbacks, `DispatchQueue`, locks, and detached tasks unless you are bridging an unavoidable system API. Concurrency style should be uniform enough that readers can reason about it locally.

## 8. Comments and Documentation

Comments should explain intent, invariants, workarounds, and external constraints. The best comments answer "why is this here?" or "what assumption must remain true?" They should not narrate obvious assignments or control flow.

Document non-obvious operational behavior near the code that depends on it. If a terminal must be restored on signal, a file format supports migration from a legacy version, or a transport requires a timeout policy, say so where that behavior is implemented.

Keep documentation close to decisions. When a module has strong architectural rules, document them in module-level docs or focused markdown instead of relying on tribal memory.

## 9. Testing and TDD

TDD is the default for logic that has a clear contract. This includes parsing, validation, state transitions, persistence rules, error mapping, configuration resolution, and deterministic orchestration. Write the failing test first, implement the smallest behavior that passes, then continue through the core failure cases before expanding to edge cases.

Not every piece of operational code is best developed test-first. For process-heavy integration code, external tool wrappers, or transport glue, it is acceptable to spike the implementation when necessary. But the behavior must still be formalized quickly with regression tests once the expected contract is known. "Too hard to test" is not an excuse to skip verification permanently.

Tests should describe behavior, not mirror implementation. A strong test asserts what the system does for a caller, not which private helper ran internally. Favor names that describe the scenario and expected result. Each test should have a single behavioral focus.

Use unit tests by default. Integration tests are for critical end-to-end workflows, component boundaries with real contracts, schema compatibility, and external tool interactions that cannot be trusted through mocks alone. Keep them narrower than the full application whenever possible.

Mocks should be shared, resettable, and owned at the boundaries you control. They should capture received inputs, expose stubbed outputs, and avoid accidental state leakage between tests. Fixtures should be minimal, deterministic, and named for the scenario they support.

Every bug fix should add a regression test unless the failure is genuinely not testable at the current boundary. If it is not testable, treat that as design feedback for future refactoring.

## 10. Coverage

Coverage is a safety metric, not a quality metric. It is useful when it tells you which important behavior is unverified. It is useless when it becomes a vanity number.

Aim for strong coverage in code that carries correctness risk: persistence and migrations, parsing and serialization, state transitions, error handling, configuration resolution, concurrency-sensitive coordination, and domain rules. Passive models, obvious wrappers, and framework glue are lower priority.

Review branch coverage, not just line coverage. The most valuable uncovered paths are usually fallback behavior, retries, cancellations, invalid inputs, and error mapping. A file can have high line coverage and still miss the cases most likely to fail in production.

Coverage targets should be set at the module or subsystem level, with stricter expectations for critical modules than for purely presentational code. New features should arrive with coverage for their core path and failure modes. Bug fixes should raise coverage around the defect, not merely preserve the global number.

If coverage is hard to add, investigate the design. Hidden side effects, large functions, and poor separation between pure logic and operational work are common reasons. Use coverage pain as a signal to improve structure over time.

## 11. Using This Guide in Refactors and Rewrites

This guide is intended to shape future work, not to invalidate the current codebase. It should be used as the standard for new modules, for files under active refactor, and for subsystems being rewritten in place. Existing code does not need to be reformatted into compliance all at once.

When touching old code, improve the boundary you are already near. Extract one pure function from side-effecting code. Replace one loose dictionary with a typed model. Move one block of persistence rules into the storage layer. Introduce one domain error that replaces a generic failure. Good refactoring is usually local and cumulative.

A codebase with modular subsystems, typed state, service boundaries, and partial concurrency discipline is usually worth improving, not restarting. The right style guide for that codebase should accelerate consistent cleanup over time. This one is meant to do that.

## 12. Concrete Swift Rules

Use `let` by default. Use `var` only when mutation is required.

Use `throws` by default for recoverable failure. Do not use `Result` unless failure must be represented as ordinary data.

Prefer `guard` for required preconditions and early exits.

Prefer enums over booleans when modeling mode or behavior.

Prefer typed constants over repeated literals.

Prefer config structs over long argument lists.

Prefer domain-specific types over tuples when the result has meaning beyond one local scope.

Prefer explicit access control. Default to `private` or `internal`. Make `public` deliberate.

Prefer `final class` unless subclassing is necessary.

Prefer small protocol surfaces. Do not create broad abstractions with weak ownership.

Prefer typed models over `[String: Any]` except at transport or parsing edges.

Prefer explicit actor isolation over informal thread assumptions.

Prefer straightforward control flow over functional chaining when the latter reduces clarity.

Prefer a short helper function over a dense block with multiple responsibilities.

Prefer comments that explain why, not what.

## 13. Closing Standard

This guide is meant to produce code that is easy to maintain under real pressure. It favors clarity, explicit boundaries, typed state, disciplined concurrency, and testable design. It does not require a pristine codebase. It requires that each new change moves the system toward better engineering rather than more accidental complexity.

If a proposed implementation is technically valid but makes ownership, failure, or behavior harder to reason about, it is not good enough. That is the bar.
