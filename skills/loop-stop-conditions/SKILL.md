---
name: loop-stop-conditions
description: Design explicit stop conditions and hard bounds for any agentic loop — the difference between a closed loop that converges predictably and an open loop that risks burning budget on confident garbage. Use when writing any loop that runs more than once without a human checking each iteration, when an agent is told to "keep going until it's good," or when a loop has no defined maximum rounds/tokens/time.
---

# Loop Engineering: Stop Conditions and Guardrails

**Source**: AI Builder Club, "Loop Engineering Guide (2026)" — practitioner synthesis citing a reported real-world incident (Uber capping agent tooling spend at $1,500/month after unattended loops consumed a year's budget in four months) and The Register's skeptical analysis of loop-autonomy claims.

## Closed loop vs. open loop

**Closed loop**: success criteria defined before execution; every iteration checked against an explicit bar; predictable token consumption; converges toward a specified target. Use when the task has a genuinely checkable definition of "done."

**Open loop**: loose conditions, wide exploration space; novel output is possible but so is slop; requires a *stronger* verifier precisely because there's more room to wander; higher token-burn risk if the verifier is weak. Use when genuine novelty is the point — but pair it with a strong verifier (see `loop-verifier-design`), never a weak one, since an open loop with a weak verifier is exactly the failure mode described below.

**Decision principle from the source**: choose based on how much novelty the task needs, weighed against how much budget risk is acceptable if the verifier turns out to be weaker than assumed.

## The failure mode this prevents

An unattended loop with a weak or absent stop condition doesn't fail loudly — it burns budget predictably, producing plausible-looking but unconverged output round after round. This is the same "doom loop" risk covered in `agentic-loop-autonomy-ladder`, viewed from the stop-condition angle rather than the autonomy-level angle.

## What a real stop condition requires

- **Explicit completion criteria** — measurable passes ("Lighthouse accessibility ≥ 95," "test suite green"), not vague satisfaction ("looks better now")
- **Hard caps** — a maximum number of rounds, a token budget, a wall-clock time limit, regardless of whether the completion criteria have been met yet
- **Provable conditions** — the verifier must be able to point to actual evidence in the agent's output, not accept a self-report of success
- **Dual control for high-stakes loops** — a separate, read-only verifier distinct from the loop's own generating agent (see `loop-verifier-design`)

## Enhancing a closed loop with bounded novelty

A closed loop doesn't have to be rigid. The source's suggested pattern: keep the hard checks as a floor, and add one exploratory instruction on top — e.g., "surprise me with the headline" — so novelty is possible *within* safety bounds, rather than choosing between "fully constrained" and "fully open" as an all-or-nothing decision.

## Practical checklist before deploying an unattended loop

- [ ] A maximum round count or token budget is set, independent of whether the task "feels" close to done
- [ ] The stop condition can be satisfied by evidence the verifier actually observes, not declared by the generating agent
- [ ] If choosing an open loop, the verifier has been deliberately made *stronger* to compensate for the wider exploration space, not left at the same strength as a closed-loop verifier would need
- [ ] Someone has asked "what does this loop cost if the stop condition never triggers and it just hits the hard cap every time" and found the answer acceptable
