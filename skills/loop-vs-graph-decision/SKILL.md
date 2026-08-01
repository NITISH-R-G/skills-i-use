---
name: loop-vs-graph-decision
description: Decide whether an agentic task should be a single loop or escalated to a multi-node graph, using a concrete 4-question test rather than defaulting to whichever pattern is trendy. Use when designing any agent system, when a "should this be multi-agent" question comes up, when a loop's single prompt is trying to do too many distinct jobs, or when reviewing whether a graph architecture is genuinely justified versus a renamed loop.
---

# Loop vs. Graph: The Decision

**Source**: AI Builder Club, "Graph Engineering Guide (2026)" and "Graph Engineering vs Loop Engineering" — synthesized practitioner content (not an academic source; the guide itself names its own inputs, including X/Twitter commentary from Peter Steinberger, @svpino, @rohit4verse, and critics like David K. Piano and Pawel Huryn who argue the underlying mechanics — directed graphs, state machines — are decades-old CS, not new capability).

## The default: try keeping it a loop first

Per the guide's own implementation checklist, step one is **"try keeping it a loop first."** A loop — discover → plan → execute → verify → repeat — handles a large fraction of agentic work. Escalating to a graph adds real, non-trivial cost: multiple prompts to maintain instead of one, a state schema to define between nodes, and new failure modes (silent state leaks between nodes, infinite routing loops, merge bugs). Reach for a graph because the task's shape demands it, not because it sounds more sophisticated.

## When a loop is the right call

- One job with a clear finish line
- Sequential steps only, no genuine parallelism
- The same tools/model throughout
- One agent can safely roam the whole task
- Self-verification (or a single verifier) is adequate for the stakes involved

## When a graph is the right call

- Work genuinely splits into distinct specialties with real handoffs
- Parallelism is needed — fan-out then join, not just sequential steps drawn as boxes
- Different models or toolsets are needed per step
- Explicit, auditable routing matters (compliance, debugging, or just wanting a readable diagram of what happens)
- Failure isolation matters — one node's failure shouldn't corrupt the whole run
- A dedicated reviewer node with real authority is needed (see `graph-reviewer-node`)

## The 4-question test before you claim you've built a graph

Before treating something as graph architecture rather than a loop wearing a costume, ask:

1. **Separate contexts?** Different prompts/tools per node — not one agent switching roles within a single transcript. *(No → it's a renamed loop.)*
2. **Real parallelism?** Something actually runs concurrently and gets merged — not sequential steps drawn as separate boxes. *(No → it's a looped flowchart.)*
3. **Readable control flow?** Routing is defined upfront as an inspectable diagram, not emerging implicitly from one agent's judgment mid-run. *(No → it's an implicit loop.)*
4. **Changed success criteria?** The objective and completion bar for the graph genuinely differ from what a loop version would use. *(No → it's a cosmetic relabeling.)*

**Scoring, per the source guide**: 0–1 yes = loop in graph clothing. 2–3 yes = genuine graph composition. 4 yes = a real architectural shift, not just new vocabulary for old mechanics.

## The one-line test

*"If you can name the specialized roles and draw arrows between them, you have a graph. If it's one job needing repetition until correct, keep it a loop."*

## Practical use

Run this test **before** writing multi-agent orchestration code, not after — the checklist step "draw edges before coding" only makes sense once you've confirmed a graph is actually warranted. If the test scores 0-1, the fix is usually to strengthen the loop's verifier (see `loop-verifier-design`) rather than to add agents.
