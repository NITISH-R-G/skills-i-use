---
name: compounding-loop-architecture
description: Architect multiple agentic loops to compound on each other via a shared file system — artifacts, contracts, and a global log — so one loop's findings can trigger another loop's work automatically, rather than running several isolated, non-communicating loops. Use when running more than one recurring agentic loop in the same project, when loops are duplicating discovery work another loop already did, or when designing the file structure a set of autonomous loops will read from and write to.
---

# Compounding Loop Architecture

**Source**: AI Builder Club, "Loop Engineering Guide (2026)," attributing this operational pattern to AI Jason's framework, with a reported concrete outcome — a multi-loop system producing 20-40 high-quality pages daily without active monitoring, via the cascading mechanism below. This is a described operational example from the source, not an independently verified benchmark — treated as an illustrative pattern, not a guaranteed result.

## The core mechanism: shared files let separate loops trigger each other

Independent loops don't need direct integration to compound — they need to read and write a **shared file system** with enough structure that one loop's output is legible input for another. The source's example: a support loop finds a bug and writes it as a signal; a separate product-growth loop reads that signal on its own schedule and turns it into a prioritized task — no direct coupling between the two loops, just a shared, well-structured file layer.

## Three file categories

**Artifacts — shared knowledge outputs.**
Category-specific folders (docs, signals, tasks, tickets, campaigns), each with a README defining its own schema and intake process, plus front-matter metadata and a modification timeline per file. **Signals** are called out specifically as the highest-leverage artifact type — capturing product ideas, friction points, and missed opportunities with source links, precisely because they're what lets one loop's incidental discovery become another loop's assigned work.

**Contracts — loop governance documents.**
One per loop, typically a `README` in that loop's own directory, stating its goal, workflow, boundaries, outstanding backlog, and timeline. Each loop reads its own contract before every execution — this is what keeps a loop's scope from drifting run over run, and what a human can read to understand what a given loop is actually supposed to do without reverse-engineering it from behavior.

**Logs — a single global work-tracking file.**
One comprehensive log across *all* loops in the system, not per-loop logs scattered separately. Entries record what was completed and when; loops read recent entries for cross-session context. A single global log is what makes it possible to reconstruct "what happened across the whole system this week" without stitching together N separate histories.

## Example multi-loop system (illustrative, from the source)

| Loop | Trigger | Function | Writes |
|---|---|---|---|
| Support | Every 30 min | Answer tickets, surface friction | Signals, engineering tasks |
| SEO | Daily, 9am | Research, publish content | Pages, conversion-gap signals |
| Product growth | Daily | Prioritize experiments | Task assignments |
| Reddit | Scheduled | Draft on-brand comments | Comment artifacts |

The cascade: the support loop's bug signal triggers the product-growth loop's task; the SEO loop's content-gap signals feed back into content generation. No loop was built aware of the others — the shared artifact structure is what makes the cascade possible.

## Practical checklist

- [ ] Every loop has its own contract file stating goal, boundaries, and backlog — read before each run, not just written once and forgotten
- [ ] Artifacts are categorized into folders with a defined schema per category, not dumped into one undifferentiated location
- [ ] A "signals" category specifically exists for incidental discoveries — friction, ideas, gaps — that aren't a given loop's primary job but might be another loop's
- [ ] One global log exists, not N per-loop logs that never get cross-referenced
- [ ] Each individual loop still has its own real verifier and stop condition (`loop-verifier-design`, `loop-stop-conditions`) — shared files coordinate *between* loops; they don't substitute for each loop being independently sound

## Where this differs from a graph

This is a looser, asynchronous coordination pattern than `graph-node-and-edge-design`'s explicit nodes/edges/state — loops here don't hand off directly or share a single state object mid-run; they coordinate indirectly through files, on their own independent schedules. Reach for a graph when nodes need tight, synchronous handoff within one run; reach for this pattern when several independently-scheduled loops benefit from seeing what the others have found.
