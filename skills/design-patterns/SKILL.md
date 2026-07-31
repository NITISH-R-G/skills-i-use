---
name: design-patterns
description: Reference for the classic Gang-of-Four object-oriented design patterns (Factory, Singleton, Proxy, Adapter, Facade, Decorator, Strategy, Observer, Template Method, Visitor) — what problem each one solves, when it earns its complexity, and when a simpler alternative is better. Use this whenever you're designing a class structure, reviewing code for a pattern opportunity, explaining why a piece of code is shaped the way it is, or the user mentions a pattern by name, asks "what pattern fits this", or describes a structural problem (swappable algorithms, notifying multiple listeners, wrapping a third-party API, one-of-a-kind object) that a pattern addresses.
---

# Design Patterns

A design pattern is a named, reusable solution to a recurring structural problem. The name is the point — "use a Strategy here" compresses a whole design conversation into two words, provided both people know what a Strategy is. That compression is also the risk: patterns get reached for because they're familiar, not because the problem calls for them, and an unearned pattern adds a layer of indirection with nothing behind it.

**Default posture**: prefer the plain, direct solution. Reach for a named pattern only when the problem it solves is actually present — a real axis of variation, not a hypothetical one. Two callers doing the same thing slightly differently is a hint; one caller is not.

## Creational — controlling how objects come into being

### Factory (Factory Method / Simple Factory)

**Problem**: calling code needs an object but shouldn't be coupled to which concrete class gets built — the choice depends on a type code, config, or runtime condition.

**Shape**: a method (or function) that takes a discriminator and returns an object typed to a common interface, hiding the `new ConcreteX()` calls behind it.

**Use it when**: you have more than one concrete implementation of an interface and the choice of which to build is itself logic worth isolating — especially if that logic would otherwise be duplicated at every call site.

**Don't use it when**: there's only one implementation. A factory that always returns the same class is ceremony with no payoff — just call the constructor.

### Singleton

**Problem**: exactly one instance of a class should exist app-wide, and everyone needs a handle to it (a logger, a connection pool, a config object).

**Shape**: the class controls its own instantiation — a private constructor plus a static accessor that creates the instance on first use and returns it thereafter.

**Use it when**: the single-instance constraint is a genuine invariant of the domain (there really is one hardware device, one global config).

**Don't use it when** — and this is most of the time it gets reached for: global mutable state makes tests order-dependent (state leaks between tests unless painstakingly reset) and hides a dependency that should be explicit. Prefer constructing one instance in composition root and passing it in (dependency injection) — you get the same "one instance" property without the global-access-point problems. Treat Singleton as the pattern most likely to be an anti-pattern in disguise.

## Structural — composing objects into larger units without new behavior

### Adapter

**Problem**: you have a class with the right behavior but the wrong interface — often a third-party or legacy type — and you need it to conform to an interface your code already depends on.

**Shape**: a thin wrapper class that implements the target interface and translates each call into the wrapped object's actual API.

**Use it when**: integrating an external library, wrapping a legacy class you can't modify, or making two independently-designed interfaces cooperate.

**Signal it's needed**: you're about to write `if (usingLibraryX) { libX.doThing() } else { libY.doTheOtherThing() }` scattered through calling code — that logic belongs in one adapter per library, behind a shared interface.

### Facade

**Problem**: a subsystem has many moving parts and a complex API surface, but most callers only need a handful of common operations.

**Shape**: one class exposing a small, high-level API that internally coordinates the subsystem's classes. The subsystem's full API still exists underneath for callers who need it.

**Use it when**: onboarding to a subsystem requires understanding five classes to do a two-line thing — write a facade with the two-line thing as a method.

**Distinction from Adapter**: Adapter makes an existing interface *fit* another; Facade makes a *complex* interface *simple*. Adapter changes shape; Facade reduces surface area.

### Decorator

**Problem**: you want to add behavior to individual objects (logging, caching, compression) without subclassing every combination, and without changing the object's class.

**Shape**: a wrapper implementing the same interface as the wrapped object, forwarding calls to it and adding behavior before/after. Decorators compose — wrap a decorator in another decorator.

**Use it when**: behavior needs to be added/removed at runtime, or the combinatorics of subclassing (`CachedLoggingCompressedStream`, `LoggingCompressedStream`, ...) would explode.

**Don't use it when**: the behavior is fixed and known at compile time for every use — a plain subclass or a straight-line function call is more legible than an unwrapping chain a reader has to trace through.

### Proxy

**Problem**: you need to control access to an object — defer its creation until first use, check permissions before forwarding a call, or add it across a process/network boundary — without the caller knowing.

**Shape**: a stand-in implementing the real object's interface, forwarding to the real object while adding the control logic (lazy init, access check, remote call marshalling).

**Use it when**: lazy loading an expensive resource, adding an authorization check transparently, or building a client-side stub for a remote service.

**Distinction from Decorator**: structurally near-identical — the difference is intent. Decorator *adds* behavior; Proxy *controls access* to the same behavior. If you're deciding between them, ask whether you're augmenting or gatekeeping.

## Behavioral — how objects communicate and share responsibility

### Strategy

**Problem**: an algorithm has several interchangeable variants (sort order, pricing rule, compression method), and the choice should be swappable independently of the code that uses it.

**Shape**: extract each variant behind a common interface; the calling class holds a reference to one and delegates to it, rather than branching on a type code internally.

**Use it when**: you catch yourself writing (or about to write) a `switch`/`if-else` chain on a type or mode flag that selects an algorithm — especially if that same switch is duplicated at more than one call site, or new variants are added regularly.

**Don't use it when**: there are two variants total and they'll never grow — an `if` is more direct than an interface plus two classes.

### Observer

**Problem**: one object's state change needs to notify an open-ended set of interested parties, without the subject knowing who they are or how many there are.

**Shape**: subjects expose subscribe/unsubscribe and call a notify method on every registered observer when state changes; observers implement a common update interface.

**Use it when**: the set of interested parties varies, grows, or is defined by a different layer than the subject (UI listening to a model, event-driven systems, pub/sub within a process).

**Watch for**: notification order dependencies (if observer B assumes observer A already ran, that's a hidden coupling the pattern was supposed to remove) and memory leaks from observers that subscribe but never unsubscribe.

### Template Method

**Problem**: several classes implement the same overall algorithm, differing only in specific steps.

**Shape**: a base class defines the algorithm as a sequence of method calls (some concrete, some abstract); subclasses override only the varying steps, inheriting the fixed skeleton.

**Use it when**: you have near-duplicate methods across sibling classes where only 1-2 steps differ — the duplication is the skeleton, not just the varying part.

**Trade-off vs. Strategy**: Template Method uses inheritance (the variation is a subclass), Strategy uses composition (the variation is an injected object). Prefer Strategy when the variation needs to be swapped at runtime or combined with other axes of variation; Template Method is simpler when there's one clear base algorithm and variants are fixed at the class level.

### Visitor

**Problem**: you need to add a new operation across a fixed hierarchy of classes (an AST, a document object model) without modifying every class in that hierarchy each time.

**Shape**: each element in the hierarchy accepts a visitor object and calls back into it (`element.accept(visitor)` → `visitor.visitConcreteElement(this)`); the operation lives in the visitor, one method per concrete element type.

**Use it when**: the *set of element types* is stable but the *set of operations* over them grows (a compiler AST that gets a new analysis pass added periodically, but rarely gets a new node type).

**Don't use it when** the reverse is true — element types are added frequently and operations are stable. Visitor inverts the usual extension axis; picking it when types churn means touching every visitor implementation on every new type, which is worse than what you started with. This is the clearest case in this whole list where the pattern can actively hurt if applied to the wrong axis of change — check which side varies before reaching for it.

## Choosing among near-neighbors

| If you need to... | Reach for |
|---|---|
| Build the right subclass based on a runtime condition | Factory |
| Make an incompatible interface fit | Adapter |
| Simplify a complex subsystem's surface | Facade |
| Add behavior to specific instances, stackably | Decorator |
| Control/gate access to an object transparently | Proxy |
| Swap an algorithm at runtime | Strategy |
| Notify an open set of listeners on state change | Observer |
| Share a fixed skeleton, vary specific steps, via subclassing | Template Method |
| Add new operations to a stable set of types without editing them | Visitor |

## The check before applying any of these

Before naming a pattern in a design or review comment, verify:

1. **The variation is real, not hypothetical.** "We might need another payment provider someday" is not the same as having two payment providers today. Patterns pay for themselves when the variation already exists or is imminently and concretely planned — not as speculative future-proofing (see YAGNI).
2. **The pattern reduces code, or at least reduces the cost of the next change** — not just adds a layer of names on top of what a direct implementation already did clearly.
3. **The reader benefit is real.** A named pattern is a shared vocabulary shortcut *only* if the reader also knows the pattern. In an unfamiliar codebase or team, a well-named direct implementation can communicate better than a correctly-applied pattern with an unfamiliar name.

When reviewing code that already uses a pattern and it doesn't feel like it's earning its keep, that's usually a sign the axis of variation it was built for never materialized — collapsing it back to a direct implementation is a legitimate refactor, not a step backward.
