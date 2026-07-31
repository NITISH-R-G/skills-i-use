---
name: static-analysis
description: Reference for what makes static analysis (linters, type checkers, security scanners, custom code-pattern checks) actually effective in a workflow versus ignored noise — placement in the dev loop, signal-to-noise discipline, and suggested-fix design. Use this whenever the user is setting up or configuring a linter/static analyzer, complaining that static analysis findings get ignored, deciding where in CI a check should run, or asking how to write a custom lint rule people will actually act on.
---

# Static Analysis

Static analysis finds real bugs before any test runs — but only if developers actually act on what it reports. A large fraction of static analysis tooling in practice is technically correct and practically ignored, because the tool optimized for finding issues rather than for getting them fixed. Effectiveness here is a workflow-design problem as much as a detection-algorithm problem.

## The core failure mode: alert fatigue

**Mechanism**: a check that fires on things that don't actually matter (overly pedantic style nits, correct code flagged by an imprecise rule) trains developers to skim past or dismiss analysis output wholesale — including the genuine, important findings mixed in with the noise. Once a team has learned to ignore a tool's output as a matter of habit, fixing the noise doesn't immediately restore trust; the habit of ignoring outlives the specific cause.

**The asymmetric cost of false positives vs. false negatives here**: a missed bug (false negative) costs you nothing extra beyond what you'd have paid without the tool. A false positive costs the tool credibility on every future finding, correct or not — which is why a static analysis rule with even a modest false-positive rate can do net-negative work despite catching real issues, once the fatigue sets in. When tuning a rule's aggressiveness, weigh this asymmetry explicitly rather than just maximizing recall.

## Where a check belongs in the workflow — placement is a design decision

| Placement | Feedback latency | Fits |
|---|---|---|
| **In-editor, as you type** | Instant | Syntax errors, obvious type errors, the cheapest/highest-confidence checks |
| **Pre-commit / presubmit hook** | Seconds | Formatting, lint rules, fast type checks — cheap enough to run on every attempted commit without becoming a bottleneck |
| **CI, on every push** | Minutes | Anything too slow for presubmit but still needed before merge — deeper analysis, cross-file checks |
| **Scheduled / batch, not blocking** | Hours to days | Expensive whole-codebase scans, security audits, deprecation-usage sweeps — findings go to a dashboard or backlog, not a blocked merge |

**The general principle**: the more confident and cheaper a check is, the earlier it belongs in the loop — cheap, high-confidence findings caught in-editor cost the developer nothing (they haven't context-switched away yet); the same finding caught in CI ten minutes after a push costs a context switch back into code they've mentally moved on from. Expensive or lower-confidence checks (this pattern is *probably* a problem, worth a human look) belong later, as advisory findings rather than blocking gates — blocking a merge on a check with real false positives is exactly how the alert-fatigue spiral starts.

## Suggested fixes — closing the loop, not just reporting

**The single highest-leverage feature a static analysis tool can have**: alongside "here's the problem," offer "here's the fix, applied automatically or with one click." A finding with no fix attached requires the developer to understand the rule, figure out the correct fix themselves, and apply it manually — friction multiplied across every instance of that finding across a codebase. A finding with an automated fix (or even a suggested diff to review and accept) removes almost all of that friction.

**Why this matters more than detection accuracy for adoption**: a tool that finds 100 issues with no fixes gets a fraction of them actually addressed, because fixing is the expensive part, not finding. A tool that finds 60 issues with one-click fixes for 50 of them gets far more total issues resolved, even though it found fewer. Optimize the tool for fix-application rate, not just finding count, when deciding what checks are worth building or turning on.

## Custom rules — earning the right to add one

Before adding a custom lint rule to a codebase, check that it clears a real bar — an easy trap is codifying a personal style preference as a blocking rule that then fires on every PR:

1. **Does it catch a real, recurring bug class**, not just a stylistic preference? "This pattern has caused N real incidents" is a much stronger justification than "I find this pattern harder to read."
2. **Is the false-positive rate low enough to trust?** A rule that's right 95% of the time but wrong 5% of the time, applied across a large codebase, produces a lot of false positives in absolute terms — worth pilot-testing on real code before turning it on broadly, not just reasoning about it abstractly.
3. **Does it have (or can it have) an automated fix?** If not, factor the friction of manual fixes into whether it's worth adding at all — see above.
4. **Is there a documented exception path?** Some flagged code is legitimately fine (a rule against a pattern that's usually wrong but occasionally intentional) — a rule with no way to suppress it for a specific, justified case forces either ignoring the tool or contorting the code to satisfy it. A clear, auditable suppression mechanism (a comment with a required justification, reviewed like any other code) is healthier than either extreme.

## Integrating with the review process

Static analysis findings surfaced *during code review* (inline on the diff, alongside human comments) get more attention than findings that only appear in a separate dashboard or a post-merge report — they're in the same context the reviewer and author are already looking at, at the point where fixing them is cheapest (before merge, not as a follow-up). Where possible, feed findings into the same interface as human review comments rather than a separate system developers have to remember to check — this pairs directly with the `code-review` skill's Standards axis, which is exactly the kind of check static analysis can pre-populate before a human reviewer even looks.

## Practical checklist for auditing an existing static analysis setup

- Are checks blocking (pre-merge gate) placed where their speed and confidence actually justify blocking, or is something slow/noisy sitting in the fast path?
- What's the false-positive rate on the highest-volume rules — has anyone actually measured it, or is it assumed to be fine?
- What fraction of findings have automated fixes available versus requiring manual triage?
- Is there evidence of alert fatigue — findings being dismissed in bulk, or a rule getting disabled team-wide rather than fixed?
- Do findings show up where developers are already looking (in the diff, in the editor) or in a separate place they have to remember to check?
