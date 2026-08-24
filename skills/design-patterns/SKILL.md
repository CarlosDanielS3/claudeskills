---
name: design-patterns
description: "Apply Gang of Four design patterns from the Refactoring Guru catalog. USE FOR: choosing the right creational, structural, or behavioral pattern; implementing Factory Method, Strategy, Observer, Adapter, Builder, Decorator, Facade, Command, or any of the 22 classic patterns; refactoring code to use patterns; reviewing code for pattern applicability; replacing conditionals with Strategy or State; decoupling with Observer or Mediator; wrapping with Adapter or Decorator. DO NOT USE FOR: general coding standards (use software-engineering-patterns); infrastructure provisioning; CI/CD configuration."
---

# Design Patterns

Reference: [Refactoring Guru — Design Patterns](https://refactoring.guru/design-patterns)

## When to Use

- Choosing an appropriate pattern to solve a recurring design problem
- Refactoring tangled code toward a well-known structure
- Reviewing code for pattern applicability or misuse
- Explaining pattern trade-offs in PR descriptions or ADRs
- Deciding between similar patterns (e.g., Strategy vs State, Decorator vs Proxy)

## When NOT to Use

- Do not force a pattern where a simple function or module suffices (KISS, YAGNI)
- Do not apply patterns preemptively — apply when the problem is present, not anticipated
- Do not stack multiple patterns in a single class; each pattern solves one concern

---

## Pattern Catalog

### Creational Patterns

Object creation mechanisms that increase flexibility and reuse.

| Pattern | Intent | Use When |
|---------|--------|----------|
| **Factory Method** | Define an interface for creating objects in a superclass; let subclasses decide which class to instantiate | You don't know ahead of time the exact types your code needs to produce; you want to decouple construction from usage |
| **Abstract Factory** | Produce families of related objects without specifying concrete classes | You need to create sets of related objects (e.g., UI widgets for different themes) that must be used together |
| **Builder** | Construct complex objects step by step; same construction process produces different representations | Object has many optional parameters, telescoping constructors, or requires multi-step initialization |
| **Prototype** | Copy existing objects without depending on their classes | Creating an object is expensive and a similar instance already exists; you need to clone with variations |
| **Singleton** | Ensure a class has exactly one instance with a global access point | Shared resource (config, connection pool, logger) — but prefer dependency injection over global state |

**Creational guidance:**
- Default to **Factory Method** for single-product creation; escalate to **Abstract Factory** when you need product families
- Prefer **Builder** over telescoping constructors when an object has 4+ optional fields
- Use **Singleton** sparingly — it introduces global state and hinders testability; inject the instance instead

### Structural Patterns

Assemble objects and classes into larger structures while keeping them flexible and efficient.

| Pattern | Intent | Use When |
|---------|--------|----------|
| **Adapter** | Allow objects with incompatible interfaces to collaborate | Integrating a third-party library or legacy class whose interface doesn't match your port/contract |
| **Bridge** | Split a large class into two hierarchies (abstraction + implementation) that vary independently | You have multi-dimensional variation (e.g., shape × renderer) and want to avoid a combinatorial explosion of subclasses |
| **Composite** | Compose objects into tree structures; treat individual objects and compositions uniformly | You need to represent part-whole hierarchies (menus, file systems, org charts) |
| **Decorator** | Attach new behaviors by wrapping objects in special wrapper objects | You need to add responsibilities at runtime without altering existing classes; alternative to subclassing |
| **Facade** | Provide a simplified interface to a complex subsystem | You want to hide library/framework complexity behind a clean, minimal API |
| **Flyweight** | Share common state between many objects to reduce memory usage | You have thousands of similar objects and memory is a constraint |
| **Proxy** | Provide a substitute or placeholder that controls access to another object | Lazy initialization, access control, logging, caching, or remote object access |

**Structural guidance:**
- **Adapter** wraps an existing interface to match a required one; **Decorator** wraps to add behavior — don't confuse them
- **Facade** simplifies; **Adapter** converts — choose based on whether the problem is complexity or incompatibility
- **Proxy** and **Decorator** have similar structure but different intent: Proxy controls access, Decorator adds behavior

### Behavioral Patterns

Algorithms and assignment of responsibilities between objects.

| Pattern | Intent | Use When |
|---------|--------|----------|
| **Chain of Responsibility** | Pass requests along a chain of handlers; each decides to process or forward | You have multiple handlers that could process a request and want to decouple sender from receiver (middleware pipelines, validation chains) |
| **Command** | Encapsulate a request as an object containing all information to execute it | You need to queue, log, undo, or schedule operations; decouple invoker from executor |
| **Iterator** | Traverse a collection without exposing its underlying representation | You have a complex data structure (tree, graph) and want uniform traversal |
| **Mediator** | Reduce chaotic dependencies by forcing objects to communicate through a central mediator | Multiple objects interact in complex ways; you want to centralize communication logic (event buses, chat rooms) |
| **Memento** | Save and restore an object's state without revealing implementation details | You need undo/redo, snapshots, or checkpoints |
| **Observer** | Define a subscription mechanism to notify multiple objects about events on a subject | One-to-many dependency: when one object changes, all dependents must be notified (event systems, reactive UIs) |
| **State** | Alter an object's behavior when its internal state changes (appears to change class) | Object behavior depends on state and you're replacing large `if/else` or `switch` blocks on state |
| **Strategy** | Define a family of interchangeable algorithms; select one at runtime | You have multiple ways to perform an operation and want to swap them without changing the client (sorting, pricing, validation) |
| **Template Method** | Define the skeleton of an algorithm in a superclass; let subclasses override specific steps | You have an algorithm with fixed structure but variable steps — and inheritance is acceptable |
| **Visitor** | Separate algorithms from the objects they operate on | You need to add operations to a stable class hierarchy without modifying those classes |

**Behavioral guidance:**
- **Strategy** vs **State**: Strategy swaps algorithms externally; State transitions internally. If the object itself decides when to change behavior, use State
- **Observer** vs **Mediator**: Observer is one-to-many broadcast; Mediator centralizes many-to-many communication
- **Command** vs **Strategy**: Both encapsulate behavior, but Command wraps a complete request (can undo/queue); Strategy wraps an algorithm (interchangeable)
- Prefer **Strategy** over conditionals when you have 3+ interchangeable behaviors

---

## Selection Decision Flow

```
Need to CREATE objects flexibly?
  → Single product type? → Factory Method
  → Product families?    → Abstract Factory
  → Complex construction? → Builder
  → Clone existing?      → Prototype

Need to COMPOSE or ADAPT structures?
  → Incompatible interface?  → Adapter
  → Add behavior at runtime? → Decorator
  → Simplify a subsystem?    → Facade
  → Tree structure?          → Composite
  → Control access?          → Proxy

Need to manage BEHAVIOR or COMMUNICATION?
  → Swap algorithms?         → Strategy
  → State-dependent behavior? → State
  → Notify subscribers?      → Observer
  → Undo/queue operations?   → Command
  → Pipeline of handlers?    → Chain of Responsibility
  → Centralize communication? → Mediator
```

---

## Anti-Patterns to Avoid

- **Pattern fever** — applying patterns for their own sake rather than to solve a concrete problem
- **Singleton abuse** — using Singleton as a global variable; prefer DI
- **Premature abstraction** — introducing Factory/Strategy for a single implementation; wait for the second use case
- **Deep decorator chains** — more than 2-3 layers become hard to debug; consider Composite or redesign
- **God Mediator** — a Mediator that absorbs all logic becomes a God Object; keep it thin

## Code Review Checklist

When reviewing code that uses design patterns:

1. **Necessity** — Is this pattern solving a real problem, or is it speculative?
2. **Correct pattern** — Is this the right pattern for the problem? (e.g., Strategy vs State)
3. **Simplicity** — Could a simpler approach (plain function, module) suffice?
4. **Naming** — Do class/method names reflect the pattern role? (e.g., `Handler`, `Strategy`, `Observer`)
5. **Testability** — Can each participant be tested in isolation?
6. **Documentation** — Is the pattern choice explained in the PR or a code comment at the entry point?
