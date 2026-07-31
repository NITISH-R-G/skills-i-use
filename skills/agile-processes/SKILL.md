---
name: agile-processes
description: Reference for agile software process frameworks — the Agile Manifesto's actual principles, Extreme Programming (XP) practices, Scrum ceremonies and roles, and Kanban flow — covering what problem each solves and common misapplications. Use this whenever the user asks about sprint planning, standups, backlog management, choosing between Scrum and Kanban, XP practices like pair programming, or is setting up or critiquing a team's process.
---

# Agile Processes

"Agile" names a value system (the Manifesto) that several distinct concrete frameworks (XP, Scrum, Kanban) each implement differently. Conflating the value system with any one framework — treating "doing Scrum" as synonymous with "being agile" — is the single most common process mistake; a team can run textbook Scrum ceremonies and be thoroughly un-agile in practice (heavyweight upfront planning, no actual response to change, ceremony without the underlying value).

## The Agile Manifesto — what it actually says

Four value statements, each a preference, not an absolute (the parenthetical is the manifesto's own qualifier, routinely dropped when people cite it):

> *Individuals and interactions* over processes and tools
> *Working software* over comprehensive documentation
> *Customer collaboration* over contract negotiation
> *Responding to change* over following a plan
>
> "That is, while there is value in the items on the right, we value the items on the left more."

This means process, documentation, contracts, and plans aren't discarded — they're subordinate to the left-hand items when the two conflict. A team that has stopped writing any documentation "because we're agile" has over-rotated on the letter of the statement and lost the actual point (the manifesto values documentation, just less than working software).

**The twelve principles** behind the four values are more operational than the values themselves — worth citing specifically when a process debate needs grounding rather than restating the four value pairs, which are too abstract to settle most real disagreements on their own. The ones that come up most in practice: "Working software is the primary measure of progress," "Simplicity — the art of maximizing the amount of work not done — is essential," and "At regular intervals, the team reflects on how to become more effective" (the mandate for retrospectives, present in every framework below).

## Extreme Programming (XP) — engineering practices

XP is the most concrete of the frameworks — where Scrum and Kanban mostly govern *process and flow*, XP prescribes specific *engineering* practices:

- **Pair programming** — two people, one keyboard, continuous review as code is written rather than after. Trades raw typing throughput for fewer defects reaching review/production and much faster knowledge-spreading across a team. Not free — it's genuinely more expensive in person-hours for straightforward work; the payoff is concentrated in complex, high-risk, or unfamiliar-to-the-team code, not routine changes.
- **Test-Driven Development** — write the failing test first, then the code to pass it. Covered in depth by the `tdd` skill; XP is the framework that originated the practice.
- **Continuous Integration** — integrate and test against `main` frequently (multiple times a day), rather than working on long-lived branches that diverge for weeks and merge in one large, risky event. Covered by `devops-practices`.
- **Collective code ownership** — any team member can change any part of the codebase, rather than each module having a sole owner who's the only one allowed (or willing) to touch it. Requires the shared coding standards and thorough test coverage the other XP practices produce — collective ownership without them just means everyone can break anything.
- **Simple design** — build the simplest thing that satisfies the current requirements (see YAGNI in `refactoring-catalog`'s Speculative Generality entry), trusting refactoring to handle the next requirement when it actually arrives rather than designing for it speculatively now.
- **Sustainable pace** — deliberately not "crunch as a default state." The premise is that overtime-as-normal produces more defects than it saves time, over any timeframe longer than a few days.

**When XP practices fit**: XP came out of high-uncertainty, high-defect-cost environments (its origin project was payroll software). The practices earn their overhead most clearly when the cost of a defect reaching production is high, or the domain is unfamiliar enough that pairing and TDD's tight feedback genuinely reduce risk — apply them selectively rather than as an all-or-nothing package on every team.

## Scrum — a fixed-cadence framework

**Roles**:
- **Product Owner** — owns the backlog, decides priority, represents stakeholder/customer intent. One person, one voice — a backlog with two Product Owners routinely means conflicting priorities reaching the team unresolved.
- **Scrum Master** — facilitates the process, removes blockers, protects the team's focus during the sprint. Not a manager and not a project-tracker-updater — the role is process facilitation, not task assignment.
- **Development Team** — cross-functional, self-organizing; decides *how* to build what the Product Owner prioritized.

**Ceremonies**, each with a specific purpose — running them without the purpose in mind is how they degrade into ritual:
- **Sprint Planning** — the team commits to a scoped, achievable set of backlog items for the sprint (typically 1-4 weeks). The output is a sprint goal the team believes it can hit, not just a list of tickets copied off the backlog.
- **Daily Standup** — each member states progress, plan, and blockers, in under 15 minutes. Purpose is surfacing blockers early and keeping the team synchronized — not a status report to the Scrum Master. A standup that turns into detailed problem-solving should move that discussion to a smaller follow-up, not consume the whole team's time in the ceremony itself.
- **Sprint Review** — demo the working increment to stakeholders, gather feedback. The artifact reviewed is working software, per the Manifesto's "working software is the primary measure of progress" — not a slide deck describing what was built.
- **Retrospective** — the team reflects on its own process and commits to specific, concrete improvements for the next sprint. A retro that surfaces the same complaint sprint after sprint with no resulting change is a sign the retro's outputs aren't being acted on, which quietly teaches the team retros don't matter — worth fixing before adding any other process change.

**Common misapplication**: treating the sprint boundary as a hard wall that blocks urgent legitimate work until the next sprint starts, or padding sprint commitments to guarantee 100% completion every time (which produces sandbagged estimates, not accurate ones — a team that never fails to complete a sprint is very likely under-committing, not executing perfectly).

## Kanban — a flow-based alternative

**Core mechanics**: visualize work as cards moving through columns (e.g., Backlog → In Progress → Review → Done), and — the part most often skipped — enforce **WIP limits** (a maximum number of cards allowed in each in-progress column at once).

**The WIP limit is the actual mechanism, not the board.** A kanban board without WIP limits is just a to-do list with columns — it doesn't produce any of Kanban's actual benefits. The limit forces a concrete choice when a column is full: finish something already in progress before starting something new, which is what surfaces bottlenecks (a column that's perpetually at its limit, with work backing up behind it, is telling you exactly where the process is constrained) and prevents the throughput-killing effect of too much simultaneous WIP (context-switching cost, and nothing finishing because everything's half-done).

**vs. Scrum**: Scrum batches work into fixed-length sprints with a planning/review cycle at each boundary; Kanban has no iterations — work flows continuously, pulled into the next column as capacity allows, with no equivalent to a fixed sprint boundary. This makes Kanban better suited to work with unpredictable arrival (support/ops queues, where you can't sensibly pre-plan two weeks of incoming tickets) and Scrum better suited to work that benefits from batched planning and a fixed cadence of stakeholder review (product feature development with a roadmap).

**Metrics that matter in Kanban**: **cycle time** (how long a card takes from start to done) and **throughput** (cards completed per period) are the flow-health signals — not velocity (a Scrum-specific concept measuring story points per sprint, which doesn't map onto a system with no sprints).

## Choosing a framework

| Situation | Fits better |
|---|---|
| Predictable, plannable feature work; stakeholders want a regular review cadence | Scrum |
| Unpredictable arrival of work (support, ops, maintenance); no natural batching unit | Kanban |
| High-defect-cost domain, need for tight engineering feedback loops | XP practices, layered onto either Scrum or Kanban |
| Team currently doing "Scrum" with no retro follow-through, sandbagged estimates, standups as status reports | Neither framework is broken — the ceremonies have decayed into ritual; fix the purpose behind each ceremony before considering a framework switch |

The frameworks aren't mutually exclusive — Scrumban (Scrum's cadence and ceremonies plus Kanban's WIP limits and flow metrics) is a common, legitimate hybrid, not a compromise to be embarrassed about. Optimize for the actual shape of the work and the team's actual constraints, not for framework purity.
