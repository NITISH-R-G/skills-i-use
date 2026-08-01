---
name: memory-consolidation
description: Periodically compress and promote memory — rolling old checkpoints into episode summaries, and promoting durable lessons from episodic memory into semantic memory — so the memory store stays useful and fast to read instead of growing indefinitely. Use periodically (e.g. at natural session boundaries, or when .memory/checkpoints/ or .memory/episodes/ has accumulated significantly), not as a one-time setup step.
---

# Memory Consolidation

**Source**: the Redis architecture guide names this explicitly as pipeline stage four — "consolidation decides what stays as raw episodes and what gets promoted into more durable knowledge... without it, your memory store grows indefinitely and retrieval quality degrades over time." ML Mastery's five-pattern guide independently makes the same point: "all patterns require growth management... mandatory operational components, not optional refinements."

## Two distinct consolidation moves

**1. Checkpoint → episode.** Many small, frequent checkpoints from `session-checkpoint` accumulate fast. Periodically, fold a run of checkpoints from one session into a single coherent episode entry in `.memory/episodes/` — the checkpoints did their job (safety net during the session); once the session's actually over, the individual snapshots are less useful than one narrative summary of what happened. The raw checkpoints can then be archived (see `memory-hygiene`).

**2. Episode → semantic.** When a lesson from a specific episode generalizes past its original context — "we learned that X approach doesn't work for this codebase" becomes a fact worth knowing regardless of which task surfaces it again — promote that specific, generalizable lesson into `.memory/semantic/project-facts.md`, stripped of the episode-specific narrative detail. The original episode stays in `.memory/episodes/` as the historical record of *how* that lesson was learned; the promoted version in semantic memory is the compressed, durable, always-relevant form.

## What NOT to promote

Not every episode has a semantic-memory-worthy lesson. A session that just executed a well-understood task without surprises doesn't need anything promoted — consolidating for its own sake, rather than because there's a genuine durable lesson, is exactly the kind of noise-generation this discipline exists to prevent, not produce.

## The compression discipline

When promoting, compress aggressively — semantic memory entries should read as durable, context-free facts, not a shortened version of the episode's narrative. Compare:

**Weak** (still narrative, not actually semantic): *"In the session where we worked on auth, we found that the caching layer was interfering with token refresh timing, which took a while to figure out."*

**Strong** (compressed to the durable fact): *"The caching layer's TTL must be shorter than the token refresh interval, or refreshed tokens can be served stale from cache. (Discovered: episodes/2026-08-01-auth-refactor.md)"*

## Triggering cadence

A reasonable default: run a consolidation pass at natural session boundaries (end of a session, or at the start of the next one, as part of extending `memory-bootstrap`'s orientation), or when `.memory/checkpoints/` has grown past a rough threshold (e.g., more than a session's worth of unconsolidated checkpoints sitting around). This doesn't need to be perfectly scheduled — the cost of running it a little late is degraded retrieval quality, not data loss, since nothing is deleted until `memory-hygiene`'s separate archival step.

## Relationship to memory-hygiene

Consolidation decides *what* gets compressed and promoted. `memory-hygiene` handles the resulting cleanup — archiving the now-redundant raw checkpoints, expiring old entries past a TTL, and catching conflicts. The two are sequential steps in the same maintenance pass, kept as separate skills because they're genuinely different judgment calls: consolidation asks "what's the durable lesson here," hygiene asks "what's safe to remove or flag now that consolidation is done."
