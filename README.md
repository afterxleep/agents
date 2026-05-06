# Agents

This is a repository of prompts, instructions, and skills I often use with AI coding agents.

It is a personal toolbox for getting agents to work in a consistent way across projects. Some files are broad engineering rules, some are focused prompt workflows, and some are reusable skills for specific tools, platforms, or kinds of work.

## What This Is

- **Prompts** are task-specific instructions I can paste into an agent when I want it to follow a particular workflow.
- **Instructions** are standing rules for how I want agents to write code, test changes, handle errors, and organize work.
- **Skills** are reusable, focused guides for specialized tasks. They give an agent a narrower operating mode, such as Swift concurrency work, SwiftUI refactoring, or pixel-perfect UI implementation.

## What's Here

- **[AGENTS.md](./AGENTS.md)** - General engineering standards for agents. Covers code style, testing, error handling, mocks, fixtures, dependencies, security, documentation, and git hygiene.
- **[PROJECT.md](./PROJECT.md)** - A project-specific companion file for repo-level context, structure, and workflow rules.
- **[SWIFT.md](./SWIFT.md)** - A Swift engineering guide for production codebases. It covers Swift API design, state, persistence, concurrency, testing, error handling, documentation, and related skills.
- **[PR-loop.md](./PR-loop.md)** - A prompt for autonomously triaging open GitHub PRs, handling clean merges, classifying CI failures, and escalating anything that needs human judgment through sub-PRs.
- **[skills/](./skills)** - Reusable skill definitions for focused agent workflows.

## Skills

- **[lemonsqueezy](./skills/lemonsqueezy/SKILL.md)** - Guidance for working with the LemonSqueezy REST API, including orders, subscriptions, customers, license keys, refunds, and store stats.
- **[pixel-perfect-design](./skills/pixel-perfect-design/SKILL.md)** - A workflow for implementing UI from mockups or screenshots with careful attention to spacing, typography, color, hierarchy, and visual verification.
- **[swift-concurrency](./skills/swift-concurrency/SKILL.md)** - Guidance for Swift concurrency work, including async/await, actors, tasks, `Sendable`, `@MainActor`, Swift 6 migration, and data race fixes.
- **[swiftui-performance-audit](./skills/swiftui-performance-audit/SKILL.md)** - A code-first workflow for diagnosing SwiftUI performance issues such as janky scrolling, excessive view updates, layout thrash, high CPU, and memory problems.
- **[swiftui-ui-patterns](./skills/swiftui-ui-patterns/SKILL.md)** - Practical SwiftUI patterns for building views, screens, navigation, tabs, sheets, state flow, and component composition.
- **[swiftui-view-refactor](./skills/swiftui-view-refactor/SKILL.md)** - Rules for cleaning up SwiftUI view files, structuring view code, handling dependencies, and using Observation safely.

## How I Use It

I copy or reference the relevant files when setting up an agent for a project:

- Use `AGENTS.md` for general coding behavior.
- Use `SWIFT.md` for Swift-heavy projects.
- Use a specific skill when the task needs specialized guidance.
- Use `PR-loop.md` when I want an agent to babysit pull requests.

The goal is not to make this universal or perfect. The goal is to keep the instructions explicit, reusable, and easy to change when something stops working.

## Author

Built by [Daniel Bernal](https://danielbernal.co).
