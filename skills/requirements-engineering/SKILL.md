---
name: requirements-engineering
description: Reference for capturing and scoping software requirements — user stories, use cases, MVP definition, and A/B testing for validating decisions with real usage data. Use this whenever the user is writing feature requirements, deciding what belongs in a first release/MVP, arguing about scope, choosing between user-story and use-case format, or wants to validate a product decision empirically rather than by opinion.
---

# Requirements Engineering

Requirements work exists to answer one question before code gets written: what, precisely, are we building, and for whom. The formats below are different lenses on that question — pick the one that matches the kind of ambiguity you're actually facing.

## User Stories

**Form**: `As a [role], I want [capability], so that [benefit]`.

**What it's for**: capturing intent from the user's point of view, in language a non-technical stakeholder can read and challenge. The "so that" clause is the load-bearing part — it's what lets anyone (including the implementer) judge whether a proposed solution actually satisfies the need, versus just satisfying the literal capability requested. A story with a vague or missing "so that" is a sign the actual need hasn't been understood yet — that's worth resolving before writing acceptance criteria, not after.

**Acceptance criteria**: every story needs a way to know it's done. The common shape is Given/When/Then:
```
Given [context/precondition]
When [action]
Then [observable outcome]
```
A story without acceptance criteria isn't a completed requirement — it's a placeholder for a conversation that hasn't happened yet.

**Sizing**: a good story is independently completable and demoable — this is the same "vertical slice" idea used in ticket-writing generally: it should cut through every layer needed to be verifiable on its own, not be a horizontal slice of just the database or just the UI. If a story can't be demoed without three other stories also being done, it's not actually independent — it's an artificially-split piece of one larger story, and splitting it that way just hides the real dependency instead of removing it.

**INVEST** is the standard checklist for a well-formed story: Independent, Negotiable, Valuable, Estimable, Small, Testable. Use it to critique a story draft, not as a form to fill out mechanically — a story that fails "Independent" usually needs re-slicing, not more prose.

## Use Cases

**Form**: a structured narrative of an actor interacting with a system to achieve a goal, including the main success scenario and named alternate/exception flows.

```
Use Case: Withdraw Cash
Actor: Customer
Preconditions: Customer is authenticated; account has sufficient funds
Main success scenario:
  1. Customer selects "Withdraw"
  2. Customer enters amount
  3. System verifies sufficient funds
  4. System dispenses cash
  5. System debits account
Extensions:
  3a. Insufficient funds: system displays error, returns to step 2
  4a. Dispenser jam: system logs fault, reverses any debit, alerts operator
```

**What it's for**: systems where the *sequence of interaction* and its exception paths are the actual complexity — not just "what capability exists" but "what happens at each decision point and each failure." Use cases force you to enumerate alternate flows explicitly, which user stories don't structurally demand.

**vs. User Stories**: a user story is a short statement of value, typically small enough to be one sprint's work; a use case is a full interaction script, often spanning what would be several user stories, better suited to systems with rich, branching interaction (transactional systems, workflow-heavy enterprise software) than to consumer apps built story-by-story. They're not mutually exclusive — a single use case's main scenario and each extension can each become a user story once you're ready to schedule the work.

**When to reach for use cases over stories**: the interaction has meaningfully different exception paths that each need their own design attention (payment failure, timeout, partial completion) — writing that as a flat list of independent stories tends to lose the "and if this step fails, here's what happens instead" structure that's the actual point.

## MVP (Minimum Viable Product)

**What it is, precisely**: the smallest thing you can ship that lets you test a real, specific hypothesis about the product with real users — not "the smallest version of the full product," and not "version 1 with fewer features." An MVP is scoped by *the hypothesis it tests*, not by *time available before a deadline*.

**The common misuse**: "MVP" gets used to mean "cut scope until it fits the timeline." That produces a shipped product that's just an incomplete version of the intended one — not a validated learning step. If you can't state the specific question the MVP answers ("will users pay for X" / "will users complete this flow without support" / "does this reduce churn"), what you have is a scoped-down release plan, not an MVP in the original sense.

**Practical test**: before finalizing MVP scope, write down the hypothesis and what result would prove it wrong. If nothing in the scope could disprove the hypothesis, the scope isn't testing anything — it's just smaller.

**Fake-door / concierge / Wizard-of-Oz variants**: sometimes the fastest way to test a hypothesis isn't a working feature at all — a "Buy Now" button that measures clicks before the purchase flow exists, or a human manually doing what the automated feature would eventually do. These validate demand before investing in the real implementation, and are worth considering explicitly before defaulting to "build the smallest real version."

## A/B Testing

**What it's for**: deciding between two (or more) concrete alternatives using real user behavior instead of opinion or internal debate — when the disagreement is about what users will actually do, not about a technical trade-off, an A/B test settles it with evidence rather than seniority or persuasiveness.

**Mechanics**:
1. **One variable changes** between A (control) and B (variant) — if you change the button color *and* the copy *and* the layout, a result can't be attributed to any one of them.
2. **Users are randomly assigned** to A or B, ensuring the groups are comparable on everything except the variable under test.
3. **A success metric is defined before the test runs**, not chosen afterward from whichever metric happened to move — deciding the metric post-hoc is how teams talk themselves into whatever result they wanted.
4. **Statistical significance and sample size** are checked before declaring a winner — a difference observed on a small sample can easily be noise. Stopping a test the moment it looks favorable ("peeking") inflates false positive rates; decide the sample size or duration up front and hold to it.

**What it's not for**: questions where you already know the answer from clear domain constraints (legal requirements, accessibility, security) — A/B testing settles genuine behavioral uncertainty, not decisions that are already determined by a hard constraint. Also weak for changes with delayed or hard-to-measure effects (a change that affects long-term retention won't show up in a two-week test window measuring click-through).

**Relationship to MVP**: an MVP tests whether a hypothesis is *worth pursuing at all*; A/B testing tests *which specific variant* performs better once you're already committed to the feature existing. Don't A/B test two variants of something you haven't validated anyone wants — validate the concept with an MVP-style test first, then A/B test the execution details.

## Picking the right tool

| Situation | Reach for |
|---|---|
| Small, independently shippable increment of value | User story |
| Complex interaction with meaningful branching/exception paths | Use case |
| Deciding what to build first to test a specific hypothesis | MVP scoping |
| Deciding between two known variants using real usage data | A/B test |

These compose in the natural order of a feature's life: MVP scoping decides *whether and what* to build first; use cases or user stories capture *what it does*; A/B testing refines *which variant* works best once it's live.
