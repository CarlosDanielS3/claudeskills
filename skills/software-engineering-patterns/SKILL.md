---
name: software-engineering-patterns
description: "Apply software engineering patterns and coding standards. USE FOR: writing new code, refactoring, code review, PR descriptions, commit messages, creating tests, applying SOLID, CLEAN, DDD, DRY, KISS, YAGNI patterns. Use when writing commits, reviewing code quality, enforcing naming conventions, inclusive terminology, or structuring PRs. DO NOT USE FOR: infrastructure provisioning, CI/CD pipeline setup, or deployment configuration."
---

# Software Engineering Patterns

## When to Use

- Writing or refactoring code in any language
- Creating commit messages or PR descriptions
- Reviewing code for quality and pattern adherence
- Enforcing naming conventions and inclusive terminology
- Structuring changes for clarity and maintainability

## General Standards

- **Language:** English only — all code, comments, docs, examples, commits, configs, errors, tests
- **Inclusive Terms:** allowlist/blocklist, primary/replica, placeholder/example, main branch, conflict-free, concurrent/parallel
- **Style:** Crisp academic prose with diplomatic precision; apply Socratic checks on assumptions and logic. Prefer self-documenting code over comments.
- **Brevity:** Keep every token purposeful.

## Code Patterns & Guidelines

Apply these patterns by default when creating or refactoring code. State which patterns were applied in the PR or commit message.

- **CLEAN CODE / Clean Architecture**
- **SOLID** (SRP, OCP, LSP, ISP, DIP) — always follow unless explicitly asked not to
- **DDD** (Domain-Driven Design) — always follow unless explicitly asked not to
- **DRY** (Don't Repeat Yourself)
- **KISS** (Keep It Simple)
- **YAGNI** (You Aren't Gonna Need It)
- **Meaningful naming** and small functions
- **Prefer composition over inheritance**

---

## Domain-Driven Design (DDD)

Primary sources: Eric Evans — *Domain-Driven Design* (2003); Vaughn Vernon — *Implementing Domain-Driven Design* (2013).

### Tactical Patterns — Building Blocks

| Building Block | Definition | Rules |
|----------------|------------|-------|
| **Entity** | Object with a unique identity that persists across state changes | Equality by ID, not by attributes. Contains behavior — not just getters/setters (avoid anemic models). |
| **Value Object** | Immutable object defined by its attributes, with no identity | No setters. Equality by structural comparison. Replace rather than modify. Prefer value objects over primitives for domain concepts (e.g., `EmailAddress` over `str`). |
| **Aggregate** | Cluster of entities and value objects with a consistency boundary | All access goes through the Aggregate Root. One transaction per aggregate. External references by ID only. Keep aggregates small — only include what must be transactionally consistent. |
| **Aggregate Root** | The single entity entry point for the aggregate | Enforces all invariants. Only the root has a repository. External code never reaches inner entities directly. |
| **Domain Event** | Record of something that happened in the domain | Named in past tense (`OrderPlaced`, `TranscriptProcessed`). Immutable. Used to communicate across aggregates and bounded contexts. |
| **Repository** | Abstraction over persistence for an aggregate root | Interface in domain layer, implementation in infrastructure. One repo per aggregate root. Returns domain objects, not DTOs or database rows. |
| **Domain Service** | Stateless operation that doesn't naturally belong to one entity | Contains cross-entity logic. No I/O — depends on ports for external interaction. Named using ubiquitous language. |
| **Factory** | Encapsulates complex object creation | Use when construction requires branching logic, external data, or multi-step assembly. Constructors should be simple. |
| **Specification** | Encapsulates query/filter/validation logic as a composable object | Use when predicates are complex, reused, or combined with AND/OR/NOT. |

### Aggregate Design Rules (Vernon)

1. **Design small aggregates** — default to single-entity aggregates; add children only when a true invariant requires transactional consistency.
2. **Reference other aggregates by identity** — resolve via repository when needed; never hold direct object references across aggregate boundaries.
3. **Use eventual consistency between aggregates** — domain events propagate changes; don't span transactions across aggregates.
4. **One transaction, one aggregate** — a use case should modify at most one aggregate per commit.

### Strategic Patterns — System Level

| Pattern | Purpose | When to Use |
|---------|---------|-------------|
| **Bounded Context** | Explicit boundary where a particular domain model applies | Each microservice or major module is a bounded context. Domain classes are NOT shared across contexts. |
| **Ubiquitous Language** | Shared vocabulary between developers and domain experts within a bounded context | All class names, methods, events, and variables use domain terms — not technical jargon. Different contexts may use different terms for the same real-world concept. |
| **Context Map** | Visual representation of relationships between bounded contexts | Document relationships: Conformist, Anti-Corruption Layer, Open Host Service, Published Language, Shared Kernel, Customer-Supplier. |
| **Anti-Corruption Layer (ACL)** | Translation layer between bounded contexts or external systems | External models must NOT leak into the domain. The ACL converts external DTOs to domain objects. Changes to external systems affect only the ACL adapter. |
| **Published Language** | Versioned schema for events crossing context boundaries | Use for domain events consumed by other contexts. Schema must be backward-compatible. |
| **Shared Kernel** | Small, shared model owned jointly by two contexts | Use sparingly — any change requires agreement from both teams. Prefer ACL over Shared Kernel when possible. |

### Anemic Domain Model (Anti-Pattern)

An anemic model has entities that are pure data containers (only getters/setters) with all behavior in external "service" classes. This violates DDD and OOP fundamentals:

- **Symptom**: Entity has only fields and property accessors; a separate class contains all the business logic.
- **Fix**: Move behavior into the entity. An `Order` should know how to `addItem()`, `calculateTotal()`, and `validateForCheckout()` — not delegate these to `OrderService`.
- **Exception**: Infrastructure concerns (persistence, messaging) remain in adapters — only *domain logic* belongs in entities.

---

## SOLID Principles — Detailed

Source: Robert C. Martin — *Agile Software Development, Principles, Patterns, and Practices* (2002); *Clean Architecture* (2017).

### SRP — Single Responsibility Principle

> A class should have one, and only one, reason to change — meaning it serves one actor.

- Each class/module has a single well-defined purpose.
- If describing what a class does requires "and", it likely has multiple responsibilities.
- Application handlers should not also format output, manage persistence, or send notifications.
- **Test**: If two different stakeholders could request changes to the same class for unrelated reasons, split it.

### OCP — Open-Closed Principle

> Software entities should be open for extension but closed for modification.

- Add new behavior through new classes (strategy, decorator, new adapter), not by modifying existing ones.
- Watch for growing `if/elif/switch` chains — they often signal a missing abstraction.
- Use polymorphism over conditionals for behavioral variation.
- **Test**: Can you add a new feature without editing existing, tested code?

### LSP — Liskov Substitution Principle

> Subtypes must be substitutable for their base types without altering correctness.

- Overridden methods must honor the base contract: same or weaker preconditions, same or stronger postconditions.
- A subclass must not throw unexpected exceptions the base class doesn't declare.
- If a subclass needs to disable inherited behavior (empty override, `NotImplementedError`), the hierarchy is wrong.
- **Test**: Replace every instance of the base type with any subtype — does the system still behave correctly?

### ISP — Interface Segregation Principle

> No client should be forced to depend on methods it does not use.

- Keep interfaces (ports) small and role-specific.
- Split fat interfaces into focused ones (e.g., `Readable` + `Writable` instead of `ReadWriteStore`).
- Adapters implement only the interfaces they need.
- **Test**: Does implementing this port require stub methods that do nothing? If yes, split the interface.

### DIP — Dependency Inversion Principle

> High-level modules should not depend on low-level modules. Both should depend on abstractions.

- Domain and application layers define ports (interfaces/abstract classes).
- Infrastructure provides concrete adapters.
- Constructor injection wires them at the composition root.
- Never `import` a concrete adapter in the domain or application layer.
- **Test**: Can you swap the database, message queue, or external API by changing only the composition root?

---

## Clean Code Principles — Detailed

Source: Robert C. Martin — *Clean Code: A Handbook of Agile Software Craftsmanship* (2008).

### Functions

- **Do one thing** — a function should perform a single task at a single level of abstraction.
- **Small** — aim for < 20 lines; hard limit 50. If it scrolls off screen, extract.
- **One level of abstraction per function** — don't mix high-level orchestration with low-level string manipulation.
- **Max 3 parameters** — beyond that, introduce a parameter object or builder.
- **No flag arguments** — boolean parameters that switch behavior mean the function does two things.
- **No side effects** — a function named `get` or `check` must not mutate state.
- **Command-Query Separation** — a function either changes state (command) or returns data (query), never both.
- **Prefer exceptions over error codes** at boundaries; use result types in domain logic.

### Naming

- **Reveal intent** — the name should answer: why it exists, what it does, how it's used.
- **Avoid disinformation** — don't use `accountList` if it's not a list; don't use `hp` for hypotenuse.
- **Make meaningful distinctions** — `productInfo` vs. `productData` is noise; choose one or make the distinction real.
- **Use pronounceable, searchable names** — no single-letter variables outside tiny loop counters.
- **Class names are nouns** (`Order`, `TranscriptAnalysis`), **method names are verbs** (`calculateTotal`, `processTranscript`).

### Error Handling

- Define domain-specific error types; never throw raw strings or generic `Error`.
- Fail fast at system boundaries; recover gracefully internally.
- Propagate errors with context — wrap, don't swallow.
- Use result types or discriminated unions over thrown exceptions where the language supports it.
- Never silently catch and ignore errors; at minimum, log and re-throw.
- Don't return `null` — use Optional, Maybe, or a domain-specific empty value.

### The Boy Scout Rule

> Leave the code cleaner than you found it.

In every PR, improve one small thing beyond the scope of the ticket — a better name, an extracted function, a removed dead import. Keep it within reason.

### F.I.R.S.T. Test Principles

- **Fast** — tests run in seconds, not minutes.
- **Independent** — no test depends on the result of another.
- **Repeatable** — same result in any environment.
- **Self-validating** — boolean outcome (pass/fail), no manual inspection.
- **Timely** — written with or just before the production code.

---

## Clean Architecture — Layer Model

Source: Robert C. Martin — *Clean Architecture* (2017), Chapters 20–22.

```
┌──────────────────────────────────────────────────────┐
│                 FRAMEWORKS & DRIVERS                  │
│  (Lambda, Express, CDK, Boto3, HTTP clients)         │
│  ┌────────────────────────────────────────────────┐  │
│  │           INTERFACE ADAPTERS                   │  │
│  │  (Controllers, Gateways, Presenters, Repos)    │  │
│  │  ┌──────────────────────────────────────────┐  │  │
│  │  │     APPLICATION BUSINESS RULES           │  │  │
│  │  │  (Use Cases / Application Services)      │  │  │
│  │  │  ┌────────────────────────────────────┐  │  │  │
│  │  │  │  ENTERPRISE BUSINESS RULES         │  │  │  │
│  │  │  │  (Entities, Value Objects,          │  │  │  │
│  │  │  │   Domain Services, Domain Events)   │  │  │  │
│  │  │  └────────────────────────────────────┘  │  │  │
│  │  └──────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────┘
  Dependencies point INWARD only — The Dependency Rule.
```

### Key Rules

- **The Dependency Rule** — source code dependencies point inward. Nothing in an inner circle can know anything about something in an outer circle.
- **Entities** — enterprise-wide business rules. No framework dependency.
- **Use Cases** — application-specific business rules. Orchestrate entities; define input/output ports.
- **Interface Adapters** — convert data between use-case format and external format (DB, HTTP, message queue).
- **Frameworks & Drivers** — the outermost layer. Glue code. As thin as possible.
- **The Humble Object Pattern** — at architectural boundaries, split behavior into a testable object (business logic) and a humble object (hard-to-test framework glue).

## Git Commits

Use conventional format:

```
<type>(<scope>): <subject>
```

- **type**: `feat` | `fix` | `docs` | `style` | `refactor` | `test` | `chore` | `perf`
- **Subject**: 50 chars max, imperative mood ("add" not "added"), no period
- **Small changes**: one-line commit only
- **Complex changes**: add body explaining what/why (72-char lines) and reference issues
- Keep commits atomic (one logical change) and self-explanatory
- Split into multiple commits if addressing different concerns

## Error Handling

Source: Robert C. Martin — *Clean Code* Ch. 7.

- Define domain-specific error types; never throw raw strings or generic `Error`
- Fail fast at system boundaries (user input, external APIs); recover gracefully internally
- Propagate errors with context — wrap, don't swallow
- Use result types or discriminated unions over thrown exceptions where the language supports it
- Never silently catch and ignore errors; at minimum, log and re-throw
- Don't return `null` — use Optional, Maybe, or a domain-specific empty value
- Write try/catch/finally first when handling external calls — the error flow defines scope

## Testing Standards

Source: Robert C. Martin — *Clean Code* Ch. 9.

- **Naming**: `should <expected behavior> when <condition>` (e.g., `should return 404 when resource not found`)
- **Structure**: Arrange → Act → Assert (one logical assertion per test)
- **Isolation**: Tests must not depend on execution order or shared mutable state
- **Mocking**: Mock at architectural boundaries (ports/adapters); do not mock the unit under test
- **Coverage**: New code paths require tests. Bug fixes require a regression test that fails without the fix.
- **No test logic**: No conditionals, loops, or try/catch in test bodies
- **F.I.R.S.T.**: Fast, Independent, Repeatable, Self-validating, Timely
- **Domain tests are pure**: Entity and value object tests use real objects — no mocks
- **Repository contract tests**: Verify any adapter implementation satisfies the port interface

## Code Review Criteria

Reviewers should verify:

1. **Correctness** — Does it do what the ticket/PR description says?
2. **Pattern adherence** — SOLID, DDD boundaries, clean architecture layers respected?
3. **Naming clarity** — Can you understand intent without reading the implementation?
4. **Test quality** — Are edge cases and failure paths covered?
5. **Security** — Input validated at boundaries? No secrets in code?
6. **Scope** — Is the change focused, or does it bundle unrelated work?

Feedback etiquette:
- Prefix non-blocking suggestions with `nit:` or `suggestion:`
- Request changes only for correctness, security, or architectural violations
- Approve with comments for stylistic or minor improvements

## Dependency & Coupling Rules

Source: Robert C. Martin — *Clean Architecture* Ch. 11 (DIP), Ch. 14 (Component Coupling); Evans — *DDD* Ch. 14 (Bounded Context).

- Dependencies point **inward** (infrastructure → application → domain); never the reverse
- Domain layer has **zero** external dependencies (no framework imports, no SDK calls)
- Introduce a new third-party dependency only when: (a) it solves a non-trivial problem, (b) it's actively maintained, (c) the team agrees
- Prefer thin adapters over deep framework coupling — wrap external SDKs behind ports
- Bounded contexts isolate domain models — no sharing of domain classes across contexts
- Use Anti-Corruption Layers to translate between bounded contexts or external systems
- Stable Dependency Principle: depend in the direction of stability
- Stable Abstractions Principle: stable components should be abstract

## Naming Conventions

- **Booleans**: prefix with `is`, `has`, `should`, `can` (e.g., `isActive`, `hasPermission`)
- **Events**: past tense describing what happened (e.g., `orderPlaced`, `userRegistered`)
- **Interfaces/Ports**: describe capability, not implementation (e.g., `ContentRepository`, not `S3ContentRepository`)
- **Files**: match the primary export; use kebab-case for files, PascalCase for classes
- **Functions**: verb-first, describe action (e.g., `calculateTotal`, `validateInput`)
- **Constants**: UPPER_SNAKE_CASE for true constants; camelCase for derived config

## Observability

- Use **structured logging** (JSON) with consistent fields: `timestamp`, `level`, `message`, `correlationId`
- Log levels: `ERROR` (action required), `WARN` (degraded but functional), `INFO` (business events), `DEBUG` (development only, never in production hot paths)
- Include `correlationId` / `traceId` in all log entries for request tracing
- Log at domain boundaries: handler entry/exit, external calls, failures
- Never log secrets, PII, or full request/response payloads in production

## Security Guardrails

- **Input validation**: validate and sanitize at system boundaries (API handlers, event consumers); trust internal domain objects
- **Least privilege**: IAM roles, database users, and API keys scoped to minimum required permissions
- **Secrets**: never hardcode; use environment variables or secrets managers; never log
- **Dependencies**: keep dependencies up to date; monitor for known vulnerabilities (CVEs)
- **OWASP awareness**: guard against injection (SQL, XSS, command), broken access control, and insecure deserialization
- **Output encoding**: sanitize outputs rendered in HTML, SQL, or shell contexts

## PR / Commit Checklist

1. Mention applied patterns in PR description
2. Add or update unit tests for changed behavior
3. Keep the change small and focused
4. Verify no secrets or PII in diff
5. Confirm error paths are handled and logged
6. Ensure naming follows conventions above

## Usage in Prompts

When requesting code generation or refactoring, state:

> "Apply patterns: CLEAN, SOLID, DDD, DRY, KISS, YAGNI. State which were applied in 1–2 sentences."
