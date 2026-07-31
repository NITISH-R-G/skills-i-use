---
name: microservices-patterns-and-antipatterns
description: Reference for common microservices anti-patterns and pitfalls, and how microservices differ from classic service-oriented architecture (SOA). Use this whenever the user is designing service boundaries, reviewing a microservices architecture for problems, arguing about whether something should be its own service, or asking how microservices differ from SOA/ESB architectures.
---

# Microservices Patterns and Antipatterns

Most microservices failures aren't failures of the architecture style itself — they're the operational complexity of distribution (see `software-architecture-styles`'s monolith-vs-microservices trade-off table) compounded by a specific set of recurring mistakes in how services get carved up and how they communicate. This skill catalogs those specific mistakes.

## Anti-pattern: the distributed monolith

**What it looks like**: services are physically separate (different repos, different deploys, different processes) but so tightly coupled at runtime that they can't actually be deployed or scaled independently — a change to one service routinely requires coordinated changes and simultaneous deploys of several others.

**Why it's worse than either alternative**: it pays the full operational cost of microservices (network calls, service discovery, distributed debugging, per-service on-call) while getting none of the benefit (independent deployability, independent scaling, fault isolation) that was supposed to justify that cost. A monolith at least gets simplicity in exchange for its coupling; a distributed monolith gets neither.

**How it happens**: services carved along technical lines (a "database service," a "validation service") rather than along business-capability lines, so a single business operation fans out across many services that all have to agree and deploy together. The fix is re-drawing service boundaries around business capabilities that can genuinely change independently — not around technical layers.

## Anti-pattern: chatty services (excessive synchronous calls)

**What it looks like**: fulfilling one user-facing operation requires a long chain of synchronous service-to-service calls (Service A calls B, which calls C, which calls D) — each hop adds latency, and each hop is a new point of failure the whole chain now depends on.

**Why it compounds**: latency adds linearly down the chain, but *failure probability* compounds — if each service has 99.9% availability, a chain of five synchronous calls has roughly 99.5% combined availability, worse than any individual service. The chain is only as reliable as the product of its links, not as reliable as its best or average link.

**Mitigations**: parallelize independent calls rather than chaining them sequentially where possible; consider whether some steps genuinely need synchronous confirmation or could be async (see `software-architecture-styles`'s message-oriented section); cache aggressively for data that doesn't need to be read fresh on every call; and, most fundamentally, reconsider whether the service boundaries themselves are forcing more cross-service chatter than the domain actually requires — a boundary that's crossed constantly for a single logical operation may be drawn in the wrong place.

## Anti-pattern: shared database

**What it looks like**: multiple services read from and write to the same database/schema directly, rather than each service owning its own data store and exposing access only through its API.

**Why it defeats the point of separate services**: a shared database means services are coupled through the schema even though they're deployed separately — changing a column used by two services requires coordinating both, and one service's query patterns can degrade performance for the other. This silently recreates monolith-level coupling while still paying microservices' deployment and operational overhead. It's a specific, very common instance of the distributed-monolith anti-pattern above.

**The fix**: each service owns its data exclusively; other services access it only through the owning service's API, never by querying its database directly. This is a hard constraint worth defending even when a direct query would be the "easy" shortcut for one specific case — the shortcut is exactly how shared-database coupling starts.

## Anti-pattern: no clear service ownership

**What it looks like**: a service exists that no team clearly owns, or ownership is ambiguous enough that when it breaks, resolving "whose problem is this" takes longer than actually fixing it.

**Why microservices make this worse than a monolith**: a monolith has one clear on-call rotation by default; a system decomposed into dozens of services needs deliberate ownership mapping for each one, and this doesn't happen automatically just because the code got split up. An orphaned service is a specific microservices failure mode that has no equivalent in a monolith.

**The fix**: every service needs a named owning team from the moment it's created, tracked somewhere discoverable (a service catalog, not tribal knowledge) — "we'll figure out ownership later" for a new service reliably means it never gets a clear owner at all.

## Anti-pattern: premature decomposition

**What it looks like**: splitting into microservices before the domain's actual boundaries are understood — the initial service boundaries turn out wrong once real usage patterns emerge, and now fixing them means coordinating a change across separately-deployed, separately-owned services instead of just moving code within one codebase.

**Why this is the costliest mistake on this list**: it's the direct consequence of ignoring the "monolith first" guidance in `software-architecture-styles` — internal module boundaries in a monolith are cheap to redraw (a refactor within one codebase); service boundaries, once real consumers depend on the service's API and its own datastore exists, are expensive to redraw (a cross-team migration). Getting the boundary wrong is far more expensive to fix *after* decomposition than *before* it.

## Distinguishing feature: microservices vs. classic SOA

Both decompose a system into independently deployable services — the distinction is mostly about *how services communicate and how much shared infrastructure sits between them*:

| | Classic SOA | Microservices |
|---|---|---|
| Integration | Often via a heavyweight Enterprise Service Bus (ESB) — a centralized broker handling routing, transformation, orchestration | Lightweight, direct service-to-service calls (REST/gRPC) or a simple message broker — no centralized smart middleware |
| Where business logic and orchestration lives | Frequently pushed into the ESB itself (orchestration/transformation rules live in shared middleware) | Lives inside each service — the network/broker layer stays "dumb," services stay "smart" |
| Service size | Often larger, coarser-grained ("the billing service" doing many related things) | Smaller, finer-grained, closer to a single business capability |
| Data ownership | Sometimes shared across services via the ESB or shared databases | Each service strictly owns its own data (see the shared-database anti-pattern above) |
| Governance | Often centralized — one team/committee governs the ESB and its integration contracts | Decentralized — each service team owns their service's contract independently |

**The practical implication**: "smart endpoints, dumb pipes" is the summary heuristic — microservices push intelligence into the services themselves and keep the communication layer simple, where classic SOA often centralizes intelligence into shared middleware. If a "microservices" migration ends up recreating a smart, centralized integration layer that everything routes through, that's SOA's ESB pattern under a new name, not actually the microservices style — worth naming explicitly if that's what's happening, since it changes which anti-patterns above are even relevant (a shared-database anti-pattern, for instance, is closer to expected in a classic SOA design and worth calling out only as microservices-specific).

## Practical checklist before calling something "microservices"

- Can each service be deployed independently of the others, in practice, not just in theory?
- Does each service own its own data exclusively, with no other service querying its database directly?
- Is every service's ownership named and discoverable?
- Were service boundaries drawn from a domain model that's already reasonably well understood (ideally validated inside a monolith first), or guessed at up front?
- For the critical user-facing paths, how many synchronous service hops does a single operation require, and does the combined availability math still work out?
