---
name: loop-verifier-design
description: Design the verifier for an agentic loop — the evaluation function that decides if output is good enough — since the verifier, not the generating model, is the actual bottleneck determining whether autonomous iteration produces value or expensive garbage. Use when building any loop that iterates without a human checking every step, when an agent is asked to "keep improving X" with no defined target, or when reviewing whether a loop's self-verification is trustworthy.
---

# Loop Engineering: The Verifier Is the Bottleneck

**Source**: AI Builder Club, "Loop Engineering Guide (2026)" — industry-practitioner synthesis citing Andrew Ng's June 2026 "Batch" letter and Anthropic's Claude Code documentation. **Independently corroborated by academic work**: "Verified Multi-Agent Orchestration" (arXiv 2603.11445) finds that decoupling verification from individual agent generation — an independent model assessing whether results adequately address the goal — is what produced its measured quality gains (completeness 3.1→4.2, source quality 2.6→4.1 on a 1-5 scale versus single-agent baselines). Both sources converge on the same core claim from different directions.

## The core claim

*"A loop is just a generator wired to a verifier, and the generator was never the bottleneck."* Models now iterate cheaply — generating another attempt costs little. What actually determines whether that iteration produces value is whether the verifier can tell good output from bad. A weak verifier doesn't fail loudly: it confidently approves poor output across many iterations, silently converting compute into expensive garbage.

## Don't let the agent self-verify

Stated directly in the source: *"Don't let an agent self-verify. It doesn't work well."* An agent judging its own output shares the same blind spots and failure modes that produced the output in the first place — self-verification has a structural incentive-alignment problem, not just an occasional-mistake problem.

## Verifier tiers, roughly weakest to strongest

1. **Self-verification** — the agent judges its own work. Unreliable; avoid for anything unsupervised.
2. **Rule-based checks** — automated tests, explicit measurable criteria (a Lighthouse score, a test suite passing, a schema validation). Strong when the success condition is genuinely checkable this way.
3. **Secondary-agent verification** — a separate, independent agent with its own detailed spec reviews the work, with no stake in having produced it. This is the same principle the academic VMAO paper calls "orchestration-level verification" — decoupled from the agent that generated the result.
4. **Human evaluation** — required for outer loops with real-world stakes or fuzzy, judgment-heavy success criteria no automated check can capture.

Choose the weakest tier that's actually trustworthy for the task's stakes — a rule-based check is cheaper and more reliable than a secondary-agent check whenever the success condition can genuinely be expressed as a rule.

## The reward-function reframe

Loop engineering parallels reinforcement learning in one useful way: instead of scripting every move, you define the reward — what counts as acceptable, what counts as done — and let the agent iterate toward it. The source's framing: *"Your domain knowledge — knowing what correct looks like in your problem — is the moat. The model is a commodity. The reward function is yours."* This is why verifier design, not prompt polish, is the actual differentiating skill here.

## Concrete example — weak vs. strong

**Weak** (no real verifier): *"Make this landing page better. Keep iterating."* Produces eight different hero versions with no defined notion of "better," real cost, and no guaranteed value.

**Strong** (explicit, checkable verifier):
```
Goal: improve landing-page conversion clarity
Must pass ALL:
  - Lighthouse accessibility >= 95
  - Exactly one primary CTA above the fold
  - Hero headline <= 12 words, states the value prop
  - Layout shift (CLS) < 0.1
Loop: propose change → run checks → keep only if all pass
Stop: all green OR 5 rounds attempted
```

## Practical checklist

- [ ] "Good enough" is defined in measurable terms *before* the loop starts, not left to the agent's judgment mid-run
- [ ] The verifier is a distinct component from the generator — not the same agent grading its own homework
- [ ] For anything unsupervised or high-stakes, the verifier is rule-based, a separate agent, or human — never pure self-verification
- [ ] The verifier's pass/fail criteria are things it can actually observe evidence of, not vibes it's "allowed to declare"
