---
name: documentation-practices
description: Reference for writing effective technical documentation — matching document type to purpose (reference, design doc, tutorial, conceptual), knowing your audience, and treating docs as a maintained artifact rather than a one-time write. Use this whenever the user is writing a README, design doc, API reference, tutorial, or any technical documentation, deciding what kind of doc a situation calls for, or complaining that existing docs are stale, confusing, or unused.
---

# Documentation Practices

The recurring failure in technical documentation isn't usually "no one wrote docs" — it's writing the wrong *type* of doc for the situation, or writing one that accurately describes a system that has since changed. Documentation is a maintained artifact with a lifecycle, the same as code — treating it as a one-time deliverable is the root cause of most stale docs.

## Match the document type to its actual purpose

Trying to write one document that's simultaneously a tutorial, a reference, and a design rationale produces something that serves none of those purposes well — pick the type that matches what the reader actually needs *right now*, and write separate documents for separate needs rather than one document trying to be everything.

**Reference documentation** — precise, complete, describes *what exists* (an API's parameters, a config option's valid values, a function's contract). Organized for lookup, not for linear reading — a reader arrives with a specific question and needs to find the answer fast, not read from the top. Should be generated from or kept tightly coupled to the source (docstrings, generated API docs) wherever possible, since manually-maintained reference docs drift from the actual interface the moment someone changes a parameter and forgets to update the doc alongside it.

**Design docs** — the *reasoning* behind a decision: the problem, the alternatives considered, the trade-offs, why this approach was chosen over the others. This is the document that answers "why does it work this way" months or years later, when the original author isn't in the room to explain — the biggest mistake is omitting the alternatives that were *rejected* and why; without that, a future reader can't tell whether an odd-looking design decision was deliberate (and shouldn't be casually "fixed") or accidental.

**Tutorials** — a *linear, guided path* to a specific, concrete outcome ("build a working X by following these steps"), for a reader with no prior context on this specific system. Distinct from reference docs in that a tutorial should work end-to-end if followed exactly — untested tutorial steps are the single most common way documentation actively wastes a reader's time, since a broken step early in a linear sequence blocks everything after it.

**Conceptual documentation** — the *mental model* needed before reference docs make sense (how does this system's architecture fit together, what are the core abstractions). Bridges the gap between "I've read the tutorial and can follow steps" and "I understand this well enough to reason about cases the tutorial didn't cover."

**Landing pages** — pure navigation: given a topic, point the reader to the right one of the above. Necessary once more than a handful of docs exist for one system — without a landing page, readers discover documents by luck or by asking a person, which doesn't scale as the doc set grows.

## Know your audience — the same content is wrong for two different readers

A doc written for someone integrating with your API for the first time (needs: conceptual overview, then a tutorial, then reference for the details) is wrong for someone who already knows the system and needs to look up one specific parameter (needs: reference, nothing else, fast). Writing one document trying to serve both means the newcomer gets overwhelmed by reference-level detail before they have the concepts to make sense of it, and the expert has to scroll past conceptual explanation they don't need to find the one detail they came for.

**Practical check before writing**: name the specific reader and what they already know and don't know, before drafting. "Someone on my team, who knows the codebase but not this specific subsystem" gets a different document than "an external developer with no context." If you can't name the audience precisely, that's worth resolving before writing starts — a doc aimed at "everyone" tends to actually serve no one well.

## Documentation is like code — it needs review and maintenance, not just authorship

**Review**: a design doc or reference doc benefits from the same kind of review as a code change — does it accurately describe the system, is it clear to someone without the author's context, does it omit something a reader will need. Docs that skip review accumulate the same class of problems unreviewed code does: things that were clear to the author and opaque to everyone else, and factual errors that would have been caught by a second reader.

**Maintenance — the harder, more commonly neglected half**: a doc that was accurate when written and never touched again silently becomes wrong the moment the system it describes changes. The practical fix isn't "remember to update docs" as a hopeful policy — it's making documentation changes part of the same change that makes the doc stale in the first place (a PR that changes an API's behavior should touch the doc describing that behavior in the same PR, the same way it touches the tests) and periodically auditing docs for staleness rather than assuming silence means correctness.

**When a doc can't be kept current, deprecate it explicitly** — a stale doc that's still findable and looks authoritative actively misleads readers, which is worse than no doc at all (a missing doc prompts someone to ask a person or read the code; a wrong doc gives false confidence). Mark deprecated docs clearly, with a pointer to the current source of truth, rather than leaving them to be discovered as wrong by whoever reads them next.

## The parameters of a good document, regardless of type

- **WHO it's for** — stated or obvious from context, per the audience discussion above.
- **WHAT it covers** — scoped explicitly; a doc that doesn't say what it's *not* about tends to sprawl over time as unrelated content gets appended to "the doc that's already there."
- **WHEN it was last verified accurate** — a date or version marker, so a reader can judge whether to trust it or double-check against the current system.
- **WHERE it fits** — linked from wherever a reader would naturally arrive looking for this information (the landing page, a README, an index) — a technically excellent doc that nothing links to might as well not exist.
- **WHY** the thing it documents works the way it does, for design docs specifically — the rationale is what ages best and is hardest to reconstruct after the fact, so it's the part most worth capturing deliberately rather than left implicit.

## When you need a technical writer rather than the engineer writing it themselves

Worth bringing in dedicated documentation expertise when: the audience is large and external (public API docs, a widely-used open-source project) where documentation quality directly affects adoption; the content needs to serve genuinely different audience types simultaneously (need several coordinated documents, an information architecture, not just one page); or an engineer's draft exists but needs restructuring for clarity that requires dedicated editing time the engineer doesn't have alongside their actual engineering work. For internal, narrow-audience docs (a README for your team's service, a design doc for a decision your team made), the engineer who did the work is usually the right author — they have the context a technical writer would have to acquire from scratch.
