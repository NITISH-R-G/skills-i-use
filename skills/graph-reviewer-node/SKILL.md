---
name: graph-reviewer-node
description: Design a dedicated, read-only reviewer node with real authority to reject and route work back, separate from the nodes that produce it — the graph-level equivalent of loop-verifier-design's "don't let an agent self-verify" principle, now backed by academic evidence that orchestration-level (decoupled) verification measurably improves output quality. Use when a multi-agent graph has no explicit review/verification node, when a "reviewer" node exists but can't actually reject or route work, or when designing the conditional edge that decides pass vs. loop-back.
---

# Graph Engineering: The Reviewer Node

**Source**: AI Builder Club's graph-engineering guides name "give the reviewer node authority — separate, read-only verifier" as a specific implementation checklist item. **Academic corroboration**: "Verified Multi-Agent Orchestration" (arXiv 2603.11445) implements this exact idea as "orchestration-level verification" — an independent model assessing whether the *collective* result of a multi-agent run adequately addresses the original goal, decoupled from any individual agent's self-assessment — and measures a real quality gain from it (completeness 3.1→4.2, source quality 2.6→4.1 on a 1-5 scale, versus a single-agent baseline, on 25 market research queries).

## Why this is the highest-leverage single node in most graphs

The Researcher → Writer → Reviewer starter pattern (see `graph-node-and-edge-design`) only produces a real quality gate if the Reviewer node genuinely has the two properties its name implies:

1. **Read-only** — it doesn't also produce the work it's judging. A node that both writes and reviews its own writing has the same self-verification problem `loop-verifier-design` names for single-agent loops, just relocated into a graph.
2. **Authority** — its verdict actually controls the routing edge. A "reviewer" node that produces commentary nobody's conditional edge checks is decoration, not a gate.

## What real authority looks like, concretely

The conditional edge from Reviewer needs to branch on the Reviewer's actual verdict field in the shared state — not on the Writer's own confidence, and not on an implicit judgment buried somewhere else in the graph:

```
verdict = reviewer_node(state)
if verdict.passed:
    → Ship
else:
    state.verdict = verdict          # written explicitly for Writer to read
    → Writer (loop back)
```

If the loop-back to Writer doesn't include *why* the Reviewer rejected it (the verdict's specifics, not just pass/fail), the Writer is retrying blind — which reproduces exactly the doom-loop risk `agentic-loop-autonomy-ladder` describes, just at the graph level: the same broken draft regenerated with no new information about what was actually wrong with it.

## Designing the Reviewer's own criteria

The Reviewer node needs its own explicit verifier design — everything in `loop-verifier-design` applies to it directly, since internally it's running its own generate-then-judge cycle. In particular: its pass/fail criteria should be things it can point to specific evidence for (per the academic paper's framing of "completeness" and "source quality" as *measured*, not vibes-based, dimensions), and it needs its own bounded stop condition so a Writer↔Reviewer loop-back doesn't cycle indefinitely — cap the number of revision rounds, same as any loop.

## Practical checklist

- [ ] The Reviewer node's prompt/tools are separate from the node(s) whose work it reviews — it isn't grading its own output
- [ ] The Reviewer's verdict is a structured field in the shared state, not free-text commentary a human has to interpret to know if it passed
- [ ] The conditional edge branches on that verdict field directly
- [ ] A rejected pass includes specific, actionable detail — not just "fail" — so the loop-back node has something to act on
- [ ] There's a cap on how many Writer↔Reviewer round-trips are allowed before escalating or stopping
