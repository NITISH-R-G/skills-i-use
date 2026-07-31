---
name: software-architecture-styles
description: Reference for high-level system architecture styles — MVC (and its siblings MVP/MVVM), microservices vs. monoliths, message-oriented architecture, and publish/subscribe — covering what each solves, its real costs, and how to tell which fits a given system. Use this whenever the user is deciding how to structure a new system or service, asking whether to split a monolith into microservices, designing how components communicate (sync request/response vs. async messaging vs. events), or asking to explain/justify/critique an architectural choice.
---

# Software Architecture Styles

An architecture style is a name for a recurring shape of how a system's parts are divided and how they talk to each other. Unlike a design pattern (one class relating to another), architecture styles operate at the level of whole components, services, or layers. The stakes of picking wrong are correspondingly higher — architecture decisions are the ones hardest to reverse later, which is why they deserve explicit trade-off reasoning rather than defaulting to whatever's trendy.

## MVC and its siblings (presentation-layer separation)

**Problem**: mixing data, display logic, and user-input handling in one place makes a UI-bearing system hard to test (can't test business logic without rendering a UI) and hard to change (a display tweak risks breaking business rules).

**MVC (Model-View-Controller)**:
- **Model** — the data and business logic, ignorant of how it's displayed.
- **View** — renders the model's state; ideally passive, no business logic.
- **Controller** — handles user input, updates the model, selects the view.

The defining relationship: Model has no reference to View or Controller. View may observe Model directly (classic MVC) or only be updated by Controller (variants differ here). This one-way dependency is what makes Model unit-testable without any UI infrastructure.

**MVP (Model-View-Presenter)**: View is fully passive (no direct Model access); a Presenter mediates all View↔Model interaction. Improves testability further — you can test the Presenter's logic by mocking the View interface — at the cost of more boilerplate per screen. Common in platforms where the View is hard to unit test directly (older Android, WinForms).

**MVVM (Model-View-ViewModel)**: ViewModel exposes observable state the View binds to declaratively (data-binding), rather than the Presenter pushing updates imperatively. Fits frameworks with a binding mechanism (WPF, Angular, Vue, SwiftUI). Less manual synchronization code than MVP, but the binding "magic" can obscure control flow when debugging.

**Choosing among them**: default to MVC unless the platform's idioms push otherwise. Move to MVP when View testability matters and the platform doesn't support data-binding. Move to MVVM when the platform has a strong binding mechanism — fighting it to do manual MVP-style updates is more work, not less.

## Monolith vs. microservices

**The actual trade-off** (not "microservices are modern, monoliths are legacy" — that framing is wrong and costs real money when acted on):

| | Monolith | Microservices |
|---|---|---|
| Deployment | One unit, one pipeline | Many units, many pipelines |
| Data consistency | Easy — one database, real transactions | Hard — each service owns its data, cross-service consistency needs sagas/eventual consistency |
| Team scaling | Contention on a shared codebase past a certain team size | Teams own services independently, ship on their own schedule |
| Operational cost | Low — one thing to monitor, log, deploy | High — service discovery, distributed tracing, network failure handling, per-service on-call |
| Local dev | Fast — run the whole thing on one machine | Slow — need several services running to test one flow, or heavy mocking |
| Failure modes | A bug can take down the whole app | A single service failing shouldn't take down others — but distributed failure (partial outages, cascading timeouts) is a new failure class entirely |
| Refactoring | Rename a function, IDE finds all call sites | Changing a service's API is a cross-team coordination problem |

**Default posture**: start with a monolith unless you already know, concretely, that you need independent deployment or independent scaling for specific parts of the system. The oft-cited reasons to start with microservices — "it'll scale better," "teams need independence" — are usually not true yet at the point a system is starting: you don't have the team-scaling problem or the differential-load problem on day one, and you're paying the full operational tax (service mesh, distributed tracing, eventual consistency) before you have the problem it solves.

**Real signals it's time to split**:
- Specific modules have genuinely different scaling profiles (one gets 1000x the traffic of another) and scaling the whole monolith to serve that one hot path wastes resources.
- Team size has grown to the point that deploy contention or code-ownership conflicts are a measured, recurring cost — not a hypothetical one.
- A module needs an independent release cadence for a real business reason (compliance boundary, different team's ship schedule).

**A well-modularized monolith** (clear internal module boundaries, no cross-module reaching into internals) can be split into microservices later with far less pain than starting distributed and discovering the module boundaries were wrong. Internal module boundaries are cheap to redraw; service boundaries (with their own databases and deployed APIs) are expensive to redraw. This is the practical argument for "monolith first."

## Message-oriented architecture

**Problem**: direct service-to-service calls (synchronous request/response) couple the caller to the callee's availability — if the callee is down or slow, the caller blocks or fails too. This coupling compounds across a call chain.

**Shape**: services communicate by sending messages through an intermediary (a message broker/queue) rather than calling each other directly. The sender doesn't wait for the receiver to process the message — it enqueues and moves on.

**What it buys you**:
- **Temporal decoupling** — sender and receiver don't need to be up at the same time.
- **Load leveling** — a burst of requests queues up rather than overwhelming the receiver.
- **Retry/durability** — a message can be redelivered if the receiver fails partway through, without the sender needing to know.

**What it costs**:
- Harder to reason about — the direct call graph is replaced with an indirect one; tracing "what happens when X occurs" requires following message flows, not stack traces.
- Eventual consistency — the receiver processes the message at some later point, so system state is briefly inconsistent between send and process.
- New failure modes — duplicate delivery (most brokers guarantee at-least-once, not exactly-once), out-of-order delivery, poison messages that fail repeatedly and need a dead-letter strategy.

**Use it when**: the operation doesn't need an immediate response (sending a confirmation email, updating an analytics pipeline, triggering a background job), or when you specifically need to decouple the availability of two services.

**Don't use it when**: the caller genuinely needs a synchronous answer to proceed (checking whether a credit card charge succeeded before showing a confirmation page) — messaging adds latency and complexity for a case that direct request/response handles more simply.

## Publish/subscribe

**Problem**: message-oriented architecture with a single queue still couples the sender to knowing who the receiver is (or the queue's topic/name). When *multiple, independent* consumers need to react to the same event, that coupling gets worse — the sender would need to notify each one individually.

**Shape**: publishers emit events to a named topic without knowing who's listening; any number of subscribers register interest in a topic and receive every event published to it. Publisher and subscribers are mutually unaware of each other — only the topic ties them together.

**Distinction from a plain message queue**: a queue typically delivers each message to *one* consumer (competing consumers, for load distribution); pub/sub delivers each message to *every* subscriber (fan-out, for independent reactions to the same event). Some systems (Kafka) blend both depending on consumer-group configuration — know which behavior your broker gives you before relying on it.

**Use it when**: an event needs to trigger independent reactions in multiple, decoupled parts of the system (an order-placed event triggering inventory update, email notification, and analytics — three independent subscribers, none aware of the others) and new subscribers should be addable without changing the publisher.

**Watch for**: it becomes hard to answer "what listens to this event and why" as subscribers accumulate over time with no central registry — worth deliberately documenting topic contracts (what fields an event guarantees, what each subscriber depends on) since the loose coupling that makes pub/sub powerful is the same thing that makes it easy to lose track of.

## How these compose

These aren't mutually exclusive choices — a real system typically layers them: an MVC (or MVVM) structure inside each service; several services communicating synchronously for request/response flows and via pub/sub for cross-cutting events; message queues smoothing out load-sensitive background work. The question for any single communication path in the system is narrow — "does this specific interaction need synchronous confirmation, or can it be fire-and-forget" — not "which architecture style is our system." Answer it per-interaction, not globally.
