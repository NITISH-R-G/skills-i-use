---
name: memory-recall
description: Retrieve relevant project memory before acting on a task — checking past decisions, related episodes, and semantic facts so work doesn't contradict or duplicate what's already known, following the read-before-reasoning discipline. Use before starting any non-trivial task, before proposing an architectural approach, or when something in the current task sounds like it might have come up before.
---

# Memory Recall

**Source pattern**: the read-before-reasoning, write-after-acting loop described in Redis's long-term memory architecture guide, adapted to the file substrate in [ARCHITECTURE.md](../../ARCHITECTURE.md) — this skill is the "read" half; `memory-capture` and `session-checkpoint` are the "write" half.

## When to reach for this, beyond bootstrap

`memory-bootstrap` gives a cheap, general orientation at session start. `memory-recall` is targeted — reach for it whenever the *current* task specifically touches something that might already have a decision, a past attempt, or a relevant fact recorded:

- About to propose an architectural approach → check `.memory/decisions/` for anything already decided that this would contradict or build on
- Debugging something that feels familiar → check `.memory/episodes/` for a past session that hit something similar
- About to state something as a project convention or constraint → check `.memory/semantic/project-facts.md` first, both to avoid restating what's already known and to avoid contradicting it
- A task file exists for related work → read `.memory/tasks/<slug>.md` for its current state before assuming it's untouched

## How to search, practically

The file substrate is deliberately grep-able — a targeted search across `.memory/decisions/`, `.memory/episodes/`, and `.memory/semantic/` for keywords related to the current task is often sufficient and fast. For genuinely relational questions ("what else depends on the module I'm about to change," "what decisions touched this specific component") — the kind of query flat files answer poorly — query the MCP knowledge graph (see [`mcp/README.md`](../../mcp/README.md)) instead of trying to reconstruct the relationship by reading every file.

## What "relevant" means here — don't over-retrieve

Per the Redis guide's own tradeoff data (91% latency reduction, 90% token reduction for a modest accuracy cost, using *selective* retrieval over full-context), the goal is retrieving what's actually relevant to the current task, not everything that might theoretically relate. Reading every file in `.memory/` for every task defeats the point of having a targeted recall discipline — search specifically, read what comes back, stop there unless it points to something else worth following.

## What to do with a hit

If recall surfaces a directly relevant prior decision or episode, **cite it explicitly** in the current work — "per decisions/0012, this follows the extract-service pattern already established" — rather than silently absorbing it and re-deriving the same conclusion from scratch. This makes the memory system's actual influence visible and checkable, the same discipline `orchestrate-justification-quality` (elsewhere in this skill ecosystem) asks for in a different context: cite the specific evidence, don't just gesture at "I checked."

## What to do when recall surfaces a conflict

If a past decision or fact appears to contradict what the current task needs, don't silently override it and don't silently ignore the new evidence either — flag it explicitly to the user, and route the actual resolution through `memory-hygiene`'s conflict-handling rather than making a unilateral call mid-task.
