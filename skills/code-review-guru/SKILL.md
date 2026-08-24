---
name: code-review-guru
description: "AWS cloud-specialist code review with architect-professional rigor. USE FOR: reviewing PRs, auditing code for AWS best practices, detecting race conditions and concurrency bugs, validating Clean Architecture layer boundaries, checking IAM least-privilege, verifying acceptance criteria coverage, flagging critical security and reliability issues. Use when reviewing Lambda handlers, CDK stacks, Step Functions, SQS consumers, DynamoDB access patterns, S3 operations, Redshift queries, or any AWS service integration. DO NOT USE FOR: general coding style (use software-engineering-patterns); choosing design patterns (use design-patterns); infrastructure provisioning from scratch; CI/CD pipeline setup."
---

# Code Review Guru

AWS Solutions Architect Professional-grade code review skill. Combines Clean Architecture, Clean Code, AWS Well-Architected Framework, and concurrency safety into a single review workflow.

## When to Use

- Reviewing pull requests that touch AWS service integrations
- Auditing Lambda handlers, CDK stacks, Step Functions, or event-driven pipelines
- Detecting race conditions, deadlocks, and concurrency hazards
- Validating code against acceptance criteria before approval
- Checking IAM policies, resource configurations, and security posture
- Reviewing data pipeline code (S3, SQS, Redshift, DynamoDB, EventBridge)

## When NOT to Use

- Pure frontend code with no cloud interaction
- Initial architecture design (use Architecture Advisor agent)
- Writing new code from scratch (use software-engineering-patterns skill)

---

## Review Workflow

Execute these phases in order. Halt and report findings at each phase before proceeding.

### Phase 0 — Acceptance Criteria Gate

**Before reading any code**, demand the acceptance criteria.

1. **Require AC**: Ask the author or locate the linked Jira ticket's acceptance criteria. If none exist, flag as a blocker — code cannot be reviewed without a definition of done.
2. **Parse each criterion**: Extract testable conditions (Given/When/Then or equivalent).
3. **Build a verification matrix**: Map each AC to the code paths and tests that satisfy it.

| AC # | Criterion | Code Path | Test Coverage | Verdict |
|------|-----------|-----------|---------------|---------|
| AC-1 | ... | ... | ... | PASS / FAIL / UNTESTED |

4. **Flag gaps**: Any AC without a corresponding code path or test is an open finding.
5. **Design intent check**: Verify the implementation matches the *intent* of the AC, not just the letter. Look for edge cases the AC implies but does not explicitly state.

---

### Phase 1 — Clean Architecture & DDD Compliance

Validate structural integrity against hexagonal architecture and Domain-Driven Design tactical patterns.

#### 1A. Clean Architecture Layer Rules

Source: Robert C. Martin — *Clean Architecture* (2017).

| Check | Rule | Severity |
|-------|------|----------|
| **CA-1** | Domain layer imports NOTHING external (no SDK, no framework, no infrastructure) | CRITICAL |
| **CA-2** | Application layer depends only on domain; never imports infrastructure directly | CRITICAL |
| **CA-3** | Infrastructure adapters implement ports defined in domain or application | HIGH |
| **CA-4** | Composition root is the only place where concrete adapters are wired | HIGH |
| **CA-5** | No business logic in Lambda entry points — they delegate to application handlers | HIGH |
| **CA-6** | Dependencies point inward; no outward dependency from inner layers (the Dependency Rule) | CRITICAL |
| **CA-7** | Domain entities contain no I/O, no side effects, no mutable shared state | HIGH |
| **CA-8** | Use Cases (application services) orchestrate domain objects — they do not contain domain logic themselves | HIGH |
| **CA-9** | Frameworks are details — no framework annotations or decorators on domain or application classes | MEDIUM |
| **CA-10** | Boundary interfaces (ports) are owned by the inner layer that defines the need, not the outer layer that fulfills it | HIGH |

#### 1B. Domain-Driven Design — Tactical Patterns

Source: Eric Evans — *Domain-Driven Design* (2003); Vaughn Vernon — *Implementing Domain-Driven Design* (2013).

| Check | Rule | Severity |
|-------|------|----------|
| **DDD-1** | **Entities** have identity and lifecycle — equality is by ID, not attribute values | HIGH |
| **DDD-2** | **Value Objects** are immutable and compared by structural equality; no setter methods | HIGH |
| **DDD-3** | **Aggregates** enforce a consistency boundary — external code never reaches inside an aggregate to modify child entities directly | CRITICAL |
| **DDD-4** | **Aggregate Root** is the sole entry point — all mutations go through the root's methods | CRITICAL |
| **DDD-5** | Aggregates reference other aggregates by ID only, never by direct object reference | HIGH |
| **DDD-6** | **Domain Events** capture state transitions as first-class objects (past-tense named: `OrderPlaced`, `TranscriptProcessed`) | MEDIUM |
| **DDD-7** | **Repositories** abstract persistence — one repository per aggregate root; no repository for child entities | HIGH |
| **DDD-8** | Repository interfaces live in the domain layer; implementations live in infrastructure | HIGH |
| **DDD-9** | **Domain Services** contain logic that does not naturally belong to a single entity or value object — they are stateless | MEDIUM |
| **DDD-10** | **Factories** encapsulate complex creation logic — constructors should not contain branching or I/O | MEDIUM |
| **DDD-11** | **Bounded Context** boundaries are explicit — no sharing of domain classes across contexts; use Anti-Corruption Layers (ACL) at integration points | CRITICAL |
| **DDD-12** | **Ubiquitous Language** — class names, method names, and variables use the domain language from the bounded context, not technical jargon | HIGH |
| **DDD-13** | No anemic domain model — entities must contain behavior (methods), not just data (getters/setters) | HIGH |
| **DDD-14** | **Specifications** encapsulate complex query/filter logic as first-class objects rather than scattering predicates across services | LOW |

**DDD Strategic review** (cross-service / cross-repo):

| Check | Rule | Severity |
|-------|------|----------|
| **DDD-S1** | Context Map relationships are identified (Conformist, ACL, Open Host, Published Language, Shared Kernel) | MEDIUM |
| **DDD-S2** | Anti-Corruption Layer translates between bounded contexts — no leaking of external model into domain | HIGH |
| **DDD-S3** | Shared Kernel changes require agreement from both owning teams | HIGH |
| **DDD-S4** | Events crossing bounded contexts use a Published Language (schema versioned, backward compatible) | HIGH |

#### 1C. Clean Code Checks

Source: Robert C. Martin — *Clean Code* (2008).

| Check | Rule | Severity |
|-------|------|----------|
| **CC-1** | Functions do one thing; max 20 lines preferred, hard limit 50 | MEDIUM |
| **CC-2** | Max 3 function parameters; use a parameter object beyond that | MEDIUM |
| **CC-3** | No flag arguments (boolean params that switch behavior) | MEDIUM |
| **CC-4** | Names reveal intent — no abbreviations, no generic names (`data`, `info`, `tmp`) | MEDIUM |
| **CC-5** | No side effects hidden in function names (a `get` should not mutate state) | HIGH |
| **CC-6** | Error handling is not the happy path — separate error flows from business logic | MEDIUM |
| **CC-7** | No commented-out code; use version control history instead | LOW |
| **CC-8** | DRY — duplicated logic across files must be extracted | MEDIUM |
| **CC-9** | Command-Query Separation — a function either changes state (command) or returns data (query), never both | HIGH |
| **CC-10** | One level of abstraction per function — don't mix high-level orchestration with low-level detail | MEDIUM |
| **CC-11** | The Newspaper Metaphor — public functions at the top, private helpers below; read top-down like an article | LOW |
| **CC-12** | Avoid output arguments — prefer returning a result over mutating a passed-in parameter | MEDIUM |
| **CC-13** | Prefer exceptions over error codes — but only at system boundaries; domain logic uses result types | MEDIUM |
| **CC-14** | The Boy Scout Rule — leave the code cleaner than you found it (within scope of the PR) | LOW |

#### 1D. SOLID Principles

Source: Robert C. Martin — *Agile Software Development* (2002), *Clean Architecture* (2017).

| Check | Rule | Severity |
|-------|------|----------|
| **SOLID-1** | **SRP** — A class has one reason to change; one actor owns it. Handler classes should not also format output or manage persistence. | HIGH |
| **SOLID-2** | **OCP** — Extend behavior through new classes (strategy, decorator), not by modifying existing ones. Watch for growing `if/elif` chains. | MEDIUM |
| **SOLID-3** | **LSP** — Subtypes are substitutable for their base type. Overridden methods must honor the base contract (preconditions, postconditions, invariants). | HIGH |
| **SOLID-4** | **ISP** — Interfaces are small and role-specific. A port should not force adapters to implement methods they don't need. | MEDIUM |
| **SOLID-5** | **DIP** — High-level modules depend on abstractions (ports), not concrete implementations. Constructor injection enforces this. | HIGH |

---

### Phase 2 — AWS Best Practices (Well-Architected Framework)

Review against the six pillars. Focus on operational patterns, not theoretical compliance.

#### Operational Excellence

| Check | Rule | Severity |
|-------|------|----------|
| **OPS-1** | Lambda functions have structured logging (JSON) with correlation IDs | HIGH |
| **OPS-2** | CloudWatch alarms defined for error rates and latency P99 | MEDIUM |
| **OPS-3** | X-Ray tracing enabled for distributed service calls | MEDIUM |
| **OPS-4** | Runbook or rollback plan documented for the change | MEDIUM |

#### Security

| Check | Rule | Severity |
|-------|------|----------|
| **SEC-1** | IAM policies follow least privilege — no `*` actions or `*` resources in production | CRITICAL |
| **SEC-2** | No hardcoded secrets, API keys, or credentials anywhere in code or config | CRITICAL |
| **SEC-3** | Secrets fetched from Secrets Manager or SSM Parameter Store (SecureString) | HIGH |
| **SEC-4** | S3 buckets enforce encryption at rest (SSE-S3 or SSE-KMS) and block public access | CRITICAL |
| **SEC-5** | API endpoints enforce authentication and authorization | CRITICAL |
| **SEC-6** | Input validated and sanitized at Lambda entry points (inbound adapters) | HIGH |
| **SEC-7** | KMS key policies scoped to required principals only | HIGH |
| **SEC-8** | VPC-bound Lambdas use security groups with minimal ingress/egress | MEDIUM |
| **SEC-9** | No SQL injection vectors in Redshift queries — use parameterized queries | CRITICAL |
| **SEC-10** | CloudTrail logging enabled for sensitive operations | MEDIUM |

#### Reliability

| Check | Rule | Severity |
|-------|------|----------|
| **REL-1** | SQS consumers configure DLQ with appropriate max receive count | CRITICAL |
| **REL-2** | Lambda retry policies set with backoff; no infinite retry loops | HIGH |
| **REL-3** | Idempotency enforced for event-driven handlers (duplicate messages are safe) | CRITICAL |
| **REL-4** | Circuit breaker or timeout on external HTTP calls | HIGH |
| **REL-5** | Graceful degradation — partial failure does not corrupt state | HIGH |
| **REL-6** | DynamoDB operations use condition expressions to prevent overwrites | HIGH |
| **REL-7** | Step Functions use retry/catch blocks on every task state | HIGH |
| **REL-8** | S3 operations handle eventual consistency for cross-region replication | MEDIUM |

#### Performance Efficiency

| Check | Rule | Severity |
|-------|------|----------|
| **PERF-1** | Lambda memory sized appropriately (not default 128 MB for compute-heavy work) | MEDIUM |
| **PERF-2** | Connections reused outside handler (SDK clients, DB connections at module scope) | HIGH |
| **PERF-3** | Batch operations used where available (SQS batch send, DynamoDB batch write) | MEDIUM |
| **PERF-4** | DynamoDB access patterns avoid scans; use query with partition key | HIGH |
| **PERF-5** | S3 multipart upload for objects > 100 MB | LOW |
| **PERF-6** | Lambda cold start mitigated (provisioned concurrency for latency-sensitive paths, lazy imports for Python) | MEDIUM |

#### Cost Optimization

| Check | Rule | Severity |
|-------|------|----------|
| **COST-1** | Lambda timeout set to reasonable maximum (not 15 min default for quick tasks) | MEDIUM |
| **COST-2** | CloudWatch log retention configured (not infinite) | LOW |
| **COST-3** | S3 lifecycle rules defined for transient data | LOW |
| **COST-4** | Reserved capacity or savings plans considered for steady-state workloads | LOW |

#### Sustainability

| Check | Rule | Severity |
|-------|------|----------|
| **SUS-1** | Compute right-sized — no over-provisioned resources for intermittent workloads | LOW |
| **SUS-2** | Prefer managed services over self-hosted where equivalent | LOW |

---

### Phase 3 — Concurrency & Race Condition Analysis

This is the most critical safety phase. Inspect every shared resource access path.

#### Race Conditions

| Check | Rule | Severity |
|-------|------|----------|
| **RACE-1** | No read-modify-write without atomic operations (DynamoDB condition expressions, S3 ETags, database transactions) | CRITICAL |
| **RACE-2** | No TOCTOU (Time-of-Check-to-Time-of-Use) — if you check then act, another process can intervene | CRITICAL |
| **RACE-3** | SQS message visibility timeout exceeds maximum processing time | HIGH |
| **RACE-4** | Lambda concurrent executions do not share mutable module-level state | CRITICAL |
| **RACE-5** | Step Functions parallel branches do not write to the same resource without coordination | HIGH |
| **RACE-6** | EventBridge rules with multiple targets handle out-of-order delivery | MEDIUM |
| **RACE-7** | S3 event notifications may deliver duplicates — handler must be idempotent | HIGH |

#### Deadlocks & Livelocks

| Check | Rule | Severity |
|-------|------|----------|
| **DEAD-1** | No circular waits between services (A waits for B, B waits for A) | CRITICAL |
| **DEAD-2** | DynamoDB transactions do not lock overlapping item sets across concurrent Lambdas | HIGH |
| **DEAD-3** | Redshift queries use appropriate isolation levels; long transactions do not block others | HIGH |

#### Data Consistency

| Check | Rule | Severity |
|-------|------|----------|
| **DATA-1** | Eventual consistency assumptions are documented and handled (S3, DynamoDB global tables) | HIGH |
| **DATA-2** | Cross-service data updates use saga pattern or idempotent compensation | HIGH |
| **DATA-3** | No partial writes — operations that touch multiple resources use transactions or are individually idempotent | CRITICAL |
| **DATA-4** | Ordered processing requirements use FIFO queues or sequence numbers | HIGH |

---

### Phase 4 — Critical Issue Scan

Zero-tolerance checks. Any finding here blocks approval.

| Check | Rule | Severity |
|-------|------|----------|
| **CRIT-1** | No secrets, PII, or credentials in code, comments, logs, or error messages | CRITICAL |
| **CRIT-2** | No unhandled exceptions that could crash a Lambda and lose the triggering event | CRITICAL |
| **CRIT-3** | No unbounded loops or recursion (infinite retry, recursive Lambda invocation) | CRITICAL |
| **CRIT-4** | No silent data loss — failed operations must be logged, retried, or dead-lettered | CRITICAL |
| **CRIT-5** | No hardcoded AWS account IDs, VPC IDs, or environment-specific values | CRITICAL |
| **CRIT-6** | No `eval()`, `exec()`, or dynamic code execution with untrusted input | CRITICAL |
| **CRIT-7** | No disabled security controls (SSL verification, auth checks) even in non-prod | CRITICAL |
| **CRIT-8** | No resource leaks — connections, file handles, and SDK clients properly closed | HIGH |
| **CRIT-9** | No catch-all exception handlers that swallow errors without logging or re-raising | CRITICAL |
| **CRIT-10** | No mutable default arguments in Python function signatures | HIGH |

---

### Phase 5 — Test Adequacy

Verify tests exist and cover the changes meaningfully.

Source: Robert C. Martin — *Clean Code* Ch. 9 (Unit Tests), *Clean Architecture* Ch. 28 (The Test Boundary).

| Check | Rule | Severity |
|-------|------|----------|
| **TEST-1** | Every new code path has at least one unit test | HIGH |
| **TEST-2** | Error paths and edge cases have dedicated tests | HIGH |
| **TEST-3** | Tests follow Arrange-Act-Assert with one logical assertion per test | MEDIUM |
| **TEST-4** | Mocks are at architectural boundaries (ports/adapters), not on the unit under test | MEDIUM |
| **TEST-5** | CDK stacks have synthesis tests that validate resource existence | HIGH |
| **TEST-6** | Acceptance criteria are traceable to specific test cases | HIGH |
| **TEST-7** | No test logic — no conditionals, loops, or try/catch in test bodies | MEDIUM |
| **TEST-8** | Regression test exists for any bug fix (fails without the fix) | HIGH |
| **TEST-9** | Concurrency-sensitive code has stress/race-condition tests where feasible | MEDIUM |
| **TEST-10** | **F.I.R.S.T.** — tests are Fast, Independent, Repeatable, Self-validating, Timely (written with or before the code) | MEDIUM |
| **TEST-11** | Domain entity tests use no mocks — pure domain logic is tested with real objects and value objects | HIGH |
| **TEST-12** | Aggregate invariant tests verify that invalid state transitions are rejected | HIGH |
| **TEST-13** | Repository contract tests verify that any adapter implementation satisfies the port interface | MEDIUM |

---

## Review Output Format

Structure every review using this template:

```markdown
## Code Review — [PR Title]

### Acceptance Criteria Verification
| AC # | Criterion | Status | Evidence |
|------|-----------|--------|----------|
| AC-1 | ... | PASS/FAIL/UNTESTED | file:line or test name |

### Findings

#### CRITICAL (blocks merge)
- **[RULE-ID]** Description. File: `path/to/file.py:L42`. Fix: ...

#### HIGH (should fix before merge)
- **[RULE-ID]** Description. File: `path/to/file.py:L88`. Suggestion: ...

#### MEDIUM (fix in follow-up)
- **[RULE-ID]** Description. File: `path/to/file.py:L15`. Suggestion: ...

#### LOW / nit
- **[RULE-ID]** Description. Suggestion: ...

### Architecture Assessment
- Layer compliance: PASS / FAIL (cite violations)
- AWS Well-Architected alignment: summary per relevant pillar

### Concurrency Safety
- Race conditions: NONE FOUND / list findings
- Idempotency: VERIFIED / UNVERIFIED
- Data consistency: summary

### Test Coverage
- AC coverage: X/Y criteria have tests
- Edge cases: covered / gaps noted
- Recommendation: ...

### Verdict
**APPROVE** / **APPROVE WITH COMMENTS** / **REQUEST CHANGES**
Rationale: ...
```

---

## AWS Service-Specific Checklists

### Lambda

- [ ] Handler delegates to application layer (no business logic in entry point)
- [ ] SDK clients instantiated outside handler function
- [ ] Timeout < 900s and proportional to expected execution time
- [ ] Memory sized for workload (benchmark with AWS Lambda Power Tuning)
- [ ] Environment variables for configuration; no hardcoded values
- [ ] Reserved concurrency set if downstream cannot handle burst
- [ ] DLQ or on-failure destination configured

### SQS

- [ ] DLQ configured with maxReceiveCount aligned to retry strategy
- [ ] Visibility timeout > 6× average processing time
- [ ] Batch size appropriate for processing time vs. Lambda timeout
- [ ] FIFO queue used when ordering matters (with deduplication strategy)
- [ ] Consumer is idempotent (duplicate delivery is safe)
- [ ] Partial batch failure reporting enabled (`ReportBatchItemFailures`)

### DynamoDB

- [ ] Partition key distributes access evenly (no hot partitions)
- [ ] GSI projections include only needed attributes (avoid KEYS_ONLY when queries need data)
- [ ] Condition expressions prevent blind overwrites
- [ ] TTL configured for ephemeral data
- [ ] On-demand vs. provisioned capacity justified
- [ ] Point-in-time recovery enabled for critical tables

### S3

- [ ] Bucket policy blocks public access
- [ ] Server-side encryption enabled (SSE-S3 or SSE-KMS)
- [ ] Lifecycle rules for transient objects
- [ ] Versioning enabled for critical data
- [ ] Event notifications are idempotent (may deliver duplicates)
- [ ] Cross-region replication configured if required for DR

### Step Functions

- [ ] Every task state has Retry and Catch blocks
- [ ] Timeouts set on every task (TimeoutSeconds)
- [ ] Parallel branches do not write to same resource
- [ ] Express vs. Standard workflow type justified
- [ ] State machine input/output validated with JSONPath filters

### CDK / IaC

- [ ] No hardcoded account IDs, regions, or VPC IDs
- [ ] Cross-stack references use CfnOutput/Fn::ImportValue sparingly
- [ ] Removal policies set explicitly (RETAIN for stateful, DESTROY for ephemeral)
- [ ] Tags applied consistently (team, environment, cost center)
- [ ] Stack synthesis test exists and passes
- [ ] IAM roles scoped to minimum required permissions

### Redshift

- [ ] Queries use parameterized statements (no string interpolation)
- [ ] Distribution and sort keys chosen for query patterns
- [ ] WLM queues configured for workload isolation
- [ ] COPY commands from S3 use IAM role (no embedded credentials)
- [ ] Long-running queries have timeout limits
- [ ] Concurrency scaling enabled if burst capacity needed

---

## DDD Pattern Quick Reference

Use this during review to identify whether domain building blocks are applied correctly.

```
┌─────────────────────────────────────────────────────────────┐
│                    BOUNDED CONTEXT                          │
│                                                             │
│  ┌─────────────────────────────────────────┐                │
│  │           AGGREGATE                     │                │
│  │  ┌──────────────┐  ┌────────────────┐   │                │
│  │  │  Aggregate   │  │ Child Entity   │   │  Domain Events │
│  │  │    Root      │──│ (accessed only │   │──── emitted ──►│
│  │  │  (Entity)    │  │  via root)     │   │                │
│  │  └──────────────┘  └────────────────┘   │                │
│  │         │                               │                │
│  │  ┌──────┴───────┐                       │                │
│  │  │ Value Object │ (immutable, no ID)    │                │
│  │  └──────────────┘                       │                │
│  └─────────────────────────────────────────┘                │
│         ▲                                                   │
│         │ (by ID only)                                      │
│  ┌──────┴──────┐     ┌──────────────┐   ┌───────────────┐   │
│  │ Repository  │     │ Domain       │   │   Factory     │   │
│  │ (interface) │     │ Service      │   │ (complex      │   │
│  │             │     │ (stateless)  │   │  creation)    │   │
│  └─────────────┘     └──────────────┘   └───────────────┘   │
│                                                             │
│  Ubiquitous Language governs all names                      │
└─────────────────────────────────────────────────────────────┘
```

### Aggregate Design Rules (Vernon)

1. **Small aggregates** — prefer single-entity aggregates; add children only when true invariant requires transactional consistency.
2. **Reference by identity** — aggregates reference other aggregates by ID, resolved via repository when needed.
3. **Eventual consistency between aggregates** — use domain events, not transactions spanning multiple aggregates.
4. **One transaction per aggregate** — a single use case should modify at most one aggregate per transaction.

### Anti-Corruption Layer Pattern

When integrating with external systems (third-party APIs, legacy services, other bounded contexts):

1. Define a **translation layer** that converts external models to internal domain objects.
2. External DTOs must NEVER propagate into domain or application layers.
3. The ACL adapter implements a domain-defined port.
4. Changes to the external system require changes only in the ACL, not in domain logic.

---

## Clean Architecture Layer Reference

Source: Robert C. Martin — *Clean Architecture* (2017), Chapters 20–22.

```
┌───────────────────────────────────────────────────────┐
│                    FRAMEWORKS & DRIVERS                │
│  (Lambda runtime, Express, CDK, Boto3, HTTP clients)  │
│  ┌─────────────────────────────────────────────────┐  │
│  │              INTERFACE ADAPTERS                  │  │
│  │  (Controllers, Gateways, Presenters, Repos)     │  │
│  │  ┌───────────────────────────────────────────┐  │  │
│  │  │         APPLICATION BUSINESS RULES        │  │  │
│  │  │  (Use Cases / Application Services)       │  │  │
│  │  │  ┌─────────────────────────────────────┐  │  │  │
│  │  │  │    ENTERPRISE BUSINESS RULES        │  │  │  │
│  │  │  │    (Entities, Value Objects,         │  │  │  │
│  │  │  │     Domain Services, Domain Events)  │  │  │  │
│  │  │  └─────────────────────────────────────┘  │  │  │
│  │  └───────────────────────────────────────────┘  │  │
│  └─────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────┘
  Dependencies point INWARD only. The Dependency Rule.
```

**Key principle**: Things that change for different reasons should not depend on each other. Database schemas, UI frameworks, and external APIs are *details* — they must not dictate the shape of the domain.

---

## Severity Definitions

| Severity | Definition | Action Required |
|----------|-----------|-----------------|
| **CRITICAL** | Security vulnerability, data loss risk, race condition, or architectural violation that will cause production incidents | Must fix before merge. No exceptions. |
| **HIGH** | Reliability risk, missing error handling, insufficient testing, or AWS anti-pattern that could cause incidents under load | Should fix before merge. Exceptions require tech-debt ticket. |
| **MEDIUM** | Code quality, maintainability, or performance concern that increases long-term cost | Fix in follow-up PR. Track in backlog. |
| **LOW** | Style preference, minor optimization, or documentation improvement | Author's discretion. Use `nit:` prefix. |
