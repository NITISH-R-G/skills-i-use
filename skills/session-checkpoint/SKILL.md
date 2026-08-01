---
name: session-checkpoint
description: Write a small, timestamped snapshot of current state to .memory/checkpoints/ periodically throughout a session (not just at the end), so a session that ends abruptly — credit exhaustion, crash, closed terminal — has a recent, resumable state to bootstrap from. Use after completing a meaningful unit of work, before starting something risky or long-running, and periodically during extended sessions — not only when the user asks to "save" or "checkpoint."
---

# Session Checkpoint

**Why this exists as a distinct skill from `memory-capture`**: capture writes specific events as they happen; checkpoint writes the *overall current state* on a cadence, as a safety net independent of whether every individual event got captured. Per [ARCHITECTURE.md](../../ARCHITECTURE.md)'s Layer 3, this is the concrete implementation of the ML Mastery guide's "Execution Checkpointing" pattern, adapted from graph-workflow state to project-memory state.

## What a checkpoint contains

Small and current, not a full re-summary of the project:

```markdown
# Checkpoint: 2026-08-01T14:30:00

## Currently doing
Refactoring auth middleware to extract TokenService (see decisions/0012-token-service.md)

## Just finished
Extracted refresh logic into TokenService; existing tests pass

## Next step
Write new unit tests for TokenService directly (not just via middleware integration tests)

## Open threads
- Haven't decided whether TokenService should own the refresh interval config or receive it injected
- Noticed but didn't fix: middleware still has an unrelated logging concern mixed in
```

Four sections, consistently: currently doing, just finished, next step, open threads. This is deliberately small — a checkpoint should take seconds to write, which is what makes writing it *frequently* actually sustainable.

## When to write one

- After completing any meaningful unit of work (not every single edit — a completed sub-task, not every file save)
- Before starting something risky (a large refactor, a destructive operation, anything that might eat significant time or tokens)
- On a rough time cadence during long sessions — e.g., roughly every 20-30 minutes of active work, adjusted to the task's actual rhythm rather than a rigid timer
- Whenever something feels like "if this session ended right now, I'd want the next session to know this"

## On resume

`memory-bootstrap` reads the **most recent** checkpoint file (sorted by timestamp), not a fixed "latest.md" — this is deliberate: filenames carrying real timestamps mean the checkpoint history itself is a lightweight episodic record, even though checkpoints aren't primarily *for* that (see `memory-consolidation` for when old checkpoints get folded into a proper episode).

## Checkpoint pruning

Checkpoints accumulate fast if written every 20-30 minutes across many sessions. `memory-hygiene` handles pruning old checkpoints once their content has either been superseded by newer ones or consolidated into an episode summary — don't let this skill itself worry about cleanup; its only job is writing frequently and cheaply.

## The honest limit

Frequent checkpointing drives the worst-case loss from "the whole session" down to "the last checkpoint interval" — it does not reach mathematically perfect zero-loss, because a cutoff mid-generation, before any write completes, can still lose that increment. This is stated plainly in `ARCHITECTURE.md` rather than oversold — the practical answer to "not even a single context lost" is checkpointing frequently enough that this residual gap is irrelevant in practice, not a claim that the gap is literally zero.
