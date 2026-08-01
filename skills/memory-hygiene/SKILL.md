---
name: memory-hygiene
description: Maintain the health of project memory — archiving stale entries past their useful life, catching and flagging conflicting facts, validating that memory entries are still accurate, and cleaning up redundant checkpoints after consolidation. Use periodically as a maintenance pass (paired with memory-consolidation), when memory-recall surfaces a conflict between two entries, or when project memory has grown large enough that retrieval feels noisy.
---

# Memory Hygiene

**Source framing**: both the Redis architecture guide and the ML Mastery pattern catalog name "forgetting" as the least-solved part of memory systems — "current systems are better at storing and retrieving than deciding what to safely forget" — and both insist explicit retention policy is mandatory precisely because the underlying research problem isn't solved yet. This skill is the operational discipline that substitutes for a not-yet-solved automatic answer: explicit, checkable rules rather than assuming the memory layer manages itself.

## The four jobs bundled here, and why they're one skill

Originally conceived as five separate skills (cleanup, health, consistency, diff, merge, validate) — collapsed into one, because in practice they're facets of a single maintenance pass over the same files, not independent workflows a user would invoke separately:

**1. Archival (not deletion).** Entries past a reasonable TTL — old checkpoints already folded into an episode by `memory-consolidation`, episodes old enough that nothing has referenced them in a long time — move to `.memory/archive/`, not deleted outright. This preserves the "delete when appropriate" step in the lifecycle as a deliberate, rare, explicit action distinct from routine archival.

**2. Conflict detection.** When two memory entries disagree — a semantic fact that contradicts a more recent decision, two episodes describing the same situation differently — flag it explicitly rather than silently picking one. A conflict record: *"semantic/project-facts.md says 'deploys go through staging first' but decisions/0018 changed this to direct-to-prod for hotfixes — reconcile."* Resolution requires either updating the semantic fact to reflect the newer decision, or determining the conflict is real (different rules for different situations) and making that distinction explicit rather than leaving contradictory statements both standing.

**3. Validation.** Periodically spot-check whether semantic facts are still true — a "current" architecture description that's drifted from what the code actually does is worse than no description, because it actively misleads. This is the same discipline `documentation-practices` (elsewhere in this skill ecosystem) applies to project docs generally, applied specifically to `.memory/semantic/`.

**4. Redundancy cleanup.** After `memory-consolidation` folds checkpoints into an episode, the individual raw checkpoint files that contributed nothing beyond what's now in the episode summary can be archived — this is bookkeeping, not judgment, and follows directly from consolidation having already made the judgment call.

## Provenance-aware flagging

Per `ARCHITECTURE.md`'s adopted concern: entries tagged (by `memory-capture`) as sourced from untrusted input — a fetched web page, an unreviewed file — get extra scrutiny here specifically. An unqualified "fact" with untrusted provenance that's never been validated is exactly the case this pass should catch before it silently misdirects a future session.

## Cadence

Pair with `memory-consolidation` at the same natural checkpoints (session boundaries, or when `.memory/` has grown noticeably), rather than running as an unrelated separate ritual — consolidation decides what's durable; hygiene decides what's safe to remove, flag, or archive given that decision.

## What this explicitly doesn't do

This isn't a distributed-conflict-resolution system — for the common case (two *sequential* sessions' memory disagreeing), explicit flag-and-resolve is sufficient. True concurrent-simultaneous-write conflicts between multiple agents working at the same time are a harder problem (see `ARCHITECTURE.md`'s note on why CRDT infrastructure is out of scope) that this skill doesn't attempt to solve.
