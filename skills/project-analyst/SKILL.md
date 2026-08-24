---
name: project-analyst
description: "Analyze codebases to identify project type, tech stack, existing patterns, and recommend optimal architecture and design approaches. USE FOR: starting work on any new or unfamiliar codebase, choosing architecture for a new feature, identifying existing conventions before writing code, recommending design approaches based on project context, storing learned patterns for incremental growth. Use when: analyze project, understand codebase, what patterns, which architecture, how is this structured, before implementing."
---

# Project Analyst

Systematic codebase reconnaissance and architectural recommendation skill. Always run this before writing or reviewing code in an unfamiliar codebase to ground decisions in reality rather than assumptions.

## When to Use

- Starting work on any codebase for the first time
- Before choosing an architecture or design approach for a new feature
- When the user asks "how should I build this?" or "what's the best approach?"
- Before any non-trivial implementation to understand existing conventions
- When switching between projects with different stacks or patterns

## When NOT to Use

- Trivial single-file edits where context is obvious
- When you've already analyzed the same codebase in this session

---

## Phase 1 — Codebase Reconnaissance

Gather facts before forming opinions. Execute these steps in order.

### 1A. Project Type Identification

Examine the root directory and configuration files to classify the project.

**Files to check** (in priority order):
1. `package.json`, `tsconfig.json`, `next.config.*`, `vite.config.*` → Node.js / Frontend
2. `pyproject.toml`, `setup.py`, `setup.cfg`, `requirements.txt`, `Pipfile` → Python
3. `go.mod`, `go.sum` → Go
4. `Cargo.toml` → Rust
5. `pom.xml`, `build.gradle`, `build.gradle.kts` → Java / Kotlin
6. `Gemfile` → Ruby
7. `composer.json` → PHP
8. `*.csproj`, `*.sln` → .NET / C#
9. `cdk.json`, `serverless.yml`, `template.yaml` (SAM), `terraform/` → Infrastructure-as-Code
10. `Dockerfile`, `docker-compose.yml` → Containerized
11. `Makefile`, `justfile`, `taskfile.yml` → Build automation

**Classify into one or more project types:**

| Type | Signals |
|------|---------|
| **REST API / Backend Service** | Express, FastAPI, Flask, Spring Boot, Gin, Chi, route definitions |
| **GraphQL API** | Apollo, graphql-yoga, Strawberry, schema definitions |
| **Event-Driven / Serverless** | Lambda handlers, SQS consumers, EventBridge rules, Step Functions |
| **CLI Tool** | argparse, click, cobra, clap, commander, yargs, bin entry points |
| **Library / SDK** | Exports only, no entry point, extensive public API surface |
| **Frontend SPA** | React, Vue, Angular, Svelte, component directories |
| **Full-Stack** | Combined frontend + API in one repo (Next.js, Nuxt, Remix, Rails) |
| **Data Pipeline** | ETL scripts, Airflow DAGs, dbt models, Spark jobs, Glue scripts |
| **Infrastructure** | CDK, Terraform, CloudFormation, Pulumi stacks |
| **Monorepo** | Workspace configs (lerna, nx, turborepo, pnpm workspaces), multiple packages |
| **Mobile** | React Native, Flutter, Swift, Kotlin, Expo |
| **ML / AI** | Model training scripts, notebooks, PyTorch/TensorFlow imports, MLflow |

### 1B. Tech Stack Inventory

Build a concrete inventory — don't guess:

```markdown
## Stack Profile
- **Language(s)**: [with versions if detectable]
- **Framework(s)**: [web, ORM, testing]
- **Package manager**: [npm, pnpm, yarn, pip, poetry, cargo]
- **Build tool**: [webpack, vite, esbuild, tsc, make, gradle]
- **Test framework**: [jest, pytest, vitest, go test, JUnit]
- **Linter/Formatter**: [eslint, prettier, ruff, black, golangci-lint]
- **CI/CD**: [GitHub Actions, GitLab CI, CircleCI, Jenkins]
- **Cloud provider**: [AWS, GCP, Azure, none]
- **IaC tool**: [CDK, Terraform, SAM, Serverless Framework, none]
- **Database(s)**: [DynamoDB, PostgreSQL, Redis, MongoDB, etc.]
- **Messaging**: [SQS, SNS, EventBridge, Kafka, RabbitMQ, none]
```

### 1C. Architecture Pattern Discovery

Scan directory structure and imports to identify the architecture already in use.

**Look for these patterns:**

| Pattern | Directory Signals | Import Signals |
|---------|-------------------|----------------|
| **Clean / Hexagonal Architecture** | `domain/`, `application/`, `infrastructure/`, `adapters/`, `ports/` | Inner layers never import outer |
| **MVC** | `models/`, `views/`, `controllers/` or `routes/` | Controller imports model |
| **Layered** | `services/`, `repositories/`, `controllers/`, `models/` | Top-down dependency |
| **Feature-based / Vertical Slices** | Folders by feature (`users/`, `orders/`, `payments/`) each with their own layers | Self-contained modules |
| **Flat / Scripts** | All files in root or `src/`, no clear layering | Direct imports everywhere |
| **Microservices** | Multiple service directories, each with own config | Separate deployables |
| **Modular Monolith** | Module directories with explicit public APIs, shared kernel | Controlled cross-module imports |
| **CQRS** | `commands/`, `queries/`, separate read/write models | Command/query separation |
| **Event Sourcing** | Event store, event handlers, projections | Events as source of truth |

### 1D. Convention Extraction

Identify the team's existing conventions to respect them:

1. **Naming**: file naming (kebab, snake, camel, Pascal), class/function style
2. **Directory structure**: where new files of each type should go
3. **Error handling**: custom error types? Result types? Try/catch style?
4. **Testing**: co-located tests (`*.test.ts`) or separate `tests/` directory? Fixtures? Factories?
5. **Configuration**: env vars, config files, feature flags
6. **Documentation**: JSDoc, docstrings, README-driven, ADRs
7. **Dependency injection**: manual wiring, container, framework-provided
8. **API style**: REST conventions, response envelope, error format

---

## Phase 2 — Architectural Recommendation Matrix

Based on the project type and problem at hand, recommend the optimal approach.

### Decision Matrix: Project Type → Architecture

| Project Type | Recommended Architecture | Key Patterns | Anti-Patterns to Avoid |
|-------------|-------------------------|-------------|----------------------|
| **REST API (simple CRUD)** | Layered or Feature-based | Repository, DTO mapping, validation at boundary | Over-engineering with DDD for simple CRUD |
| **REST API (complex domain)** | Clean/Hexagonal + DDD | Aggregates, Domain Events, Ports & Adapters | Anemic domain model, leaking persistence |
| **Event-Driven / Serverless** | Hexagonal per Lambda + Event-Driven | Idempotent handlers, DLQ, Saga/Choreography | Distributed monolith, missing idempotency |
| **CLI Tool** | Command pattern + Pipeline | Command, Chain of Responsibility, Builder for args | God function, global state |
| **Library / SDK** | Facade + Builder API | Facade, Builder, Strategy for extensibility | Leaking internals, breaking changes |
| **Frontend SPA** | Component architecture + State management | Container/Presenter, Observer, Composite | Prop drilling, business logic in components |
| **Full-Stack** | Separate frontend/backend concerns | BFF pattern, API contract, shared types | Tight coupling between layers |
| **Data Pipeline** | Pipeline + Strategy | Template Method, Strategy for transforms, Factory | Monolithic scripts, no error recovery |
| **Infrastructure** | Construct composition | Builder, Composite, Factory for resources | Copy-paste stacks, no abstraction |
| **Monorepo** | Package boundaries + Dependency graph | Facade per package, Published Language | Circular dependencies, implicit coupling |
| **ML / AI** | Experiment tracking + Pipeline | Strategy for models, Template for training loops | Notebook-as-production, no reproducibility |

### Complexity-Based Scaling

Not every project needs the same level of architecture. Match complexity to the problem:

```
Simple (< 5 entities, CRUD-heavy, single bounded context)
  → Layered Architecture, Repository pattern, minimal abstractions
  → Skip: DDD aggregates, domain events, CQRS

Medium (5-15 entities, some business rules, 1-3 bounded contexts)
  → Clean Architecture, DDD tactical patterns, Ports & Adapters
  → Skip: Event sourcing, CQRS unless read/write asymmetry exists

Complex (15+ entities, rich domain logic, multiple bounded contexts)
  → Full DDD (strategic + tactical), Event-Driven, CQRS if needed
  → Consider: Event sourcing for audit-critical domains

Infrastructure-Heavy (many AWS services, pipelines, integrations)
  → Hexagonal per service, Anti-Corruption Layers, Published Language
  → Consider: Saga pattern for distributed transactions
```

---

## Phase 3 — Strategic Design Thinking

Before implementing, answer these questions:

### 3A. Problem Decomposition

1. **What is the core domain?** — The part that makes this business unique and valuable
2. **What is supporting?** — Necessary but not differentiating (auth, notifications, logging)
3. **What is generic?** — Solved problems with off-the-shelf solutions (email sending, file storage)
4. **Where are the boundaries?** — What changes independently? What must be consistent?

### 3B. Trade-Off Analysis

For every architectural decision, explicitly state:

| Decision | Gains | Costs | Reversibility |
|----------|-------|-------|---------------|
| [Choice A] | ... | ... | Easy / Hard |
| [Choice B] | ... | ... | Easy / Hard |

**Prefer reversible decisions.** When two approaches are close, pick the one easier to change later.

### 3C. Fit-for-Purpose Check

Before recommending any pattern or approach, verify:

- [ ] Does the team's current codebase already solve this a different way? → **Respect existing patterns**
- [ ] Is this pattern justified by current complexity, or only anticipated complexity? → **YAGNI**
- [ ] Can a simpler approach work for the next 6 months? → **KISS**
- [ ] Will this pattern be understood by the team maintaining it? → **Readability over cleverness**

---

## Phase 4 — Incremental Learning Protocol

The agent grows smarter with each project interaction. Use repo memory to persist discoveries.

### What to Record

After analyzing a codebase, store these findings in repo memory:

1. **Project conventions** — naming, directory structure, error handling style
2. **Verified build/test commands** — exact commands that work, not guesses
3. **Architectural decisions** — patterns in use and why (if ADRs exist, reference them)
4. **Tech stack specifics** — versions, quirks, workarounds discovered
5. **Domain vocabulary** — ubiquitous language terms and their meaning in this context
6. **Anti-patterns found** — recurring issues to flag in future reviews

### When to Record

- After completing Phase 1 reconnaissance on a new codebase
- After discovering a convention not obvious from directory structure alone
- After a successful build/test run reveals the correct commands
- After the user corrects a wrong assumption about the project
- After finding a pattern that worked particularly well for this project type

### How to Record

Use the memory tool with `/memories/repo/` path:

```json
{
  "subject": "Project uses feature-based architecture",
  "fact": "Each feature (users, orders, payments) has its own domain, application, and infrastructure directories. New features should follow the same structure. See src/features/orders/ as the canonical example.",
  "citations": ["src/features/orders/", "src/features/users/"],
  "reason": "Discovered during codebase reconnaissance. This convention is not documented but consistently applied across all 8 existing features.",
  "category": "architecture"
}
```

### Categories for Facts

- `architecture` — structural patterns and layer conventions
- `conventions` — naming, formatting, file organization
- `build` — verified build, test, lint, deploy commands
- `domain` — ubiquitous language, bounded contexts, key aggregates
- `tech-stack` — framework versions, configuration quirks
- `anti-patterns` — recurring issues to watch for
- `patterns` — design patterns that work well in this codebase

---

## Output Format

After completing reconnaissance, present findings as:

```markdown
## Project Analysis

### Classification
- **Type**: [from identification matrix]
- **Complexity**: Simple / Medium / Complex / Infrastructure-Heavy
- **Architecture**: [detected pattern]
- **Maturity**: Greenfield / Early / Established / Legacy

### Stack Profile
[filled inventory from Phase 1B]

### Existing Conventions
[key conventions from Phase 1D]

### Recommendation
**Approach**: [recommended architecture/pattern]
**Rationale**: [why this fits the project type and complexity]
**Trade-offs**: [what you gain and lose]

### Patterns to Apply
- [Pattern 1] — for [specific problem]
- [Pattern 2] — for [specific problem]

### Patterns to AVOID
- [Anti-pattern] — because [reason specific to this project]

### Implementation Plan
1. [Step 1 — what to build first]
2. [Step 2 — what depends on step 1]
3. [Step 3 — ...]
```
