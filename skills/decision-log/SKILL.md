---
name: decision-log
description: Record significant architectural or design decisions to .memory/decisions/ in a consistent, ADR-style format, capturing the alternatives considered and why they were rejected — not just the choice made. Use whenever a non-trivial design or architectural decision is made, especially one with real alternatives that were seriously considered and rejected.
---

# Decision Log

**Relationship to `architecture-decision` elsewhere in this skill ecosystem**: that skill covers *authoring* a full Architecture Decision Record as a standalone practice. This skill is the same underlying discipline, specifically wired into the `.memory/` system's file layout and lifecycle — if a project already uses `architecture-decision`, point its output at `.memory/decisions/` rather than maintaining two separate decision-tracking locations.

## Why the rejected alternatives matter more than the choice

The single most valuable thing a decision record can prevent is a future session — possibly in a different tool, possibly months later — re-proposing an approach that was already tried and specifically rejected, for reasons that are no longer visible anywhere except someone's memory of a conversation that's now gone. A decision record that states only the final choice, with no record of what else was considered, doesn't actually prevent this.

## Format

```markdown
# 0012: Extract TokenService from auth middleware

## Status
Accepted

## Context
Auth middleware was handling three unrelated concerns: request validation, token refresh,
and session logging. Token refresh logic specifically needed dedicated test coverage that
was hard to get at while it was inline.

## Decision
Extract token refresh into a dedicated TokenService class, injected into the middleware
rather than implemented inline.

## Alternatives considered and rejected
- **Keep inline, extract a helper function**: reduced duplication but didn't solve the
  testability problem — the helper still required the full middleware request/response
  cycle to invoke in tests.
- **Move refresh logic to a separate middleware layer instead of a service class**:
  would have required restructuring the middleware chain order, higher-risk change for
  the same benefit a service class gets more simply.

## Consequences
- TokenService now needs its own test suite (separate from middleware integration tests)
- Open question not yet resolved: whether TokenService owns its refresh-interval config
  or receives it injected — tracked separately, see checkpoints/2026-08-01T14-30-00.md
```

## Numbering and filing

Sequential numbers (`0001`, `0002`, ...) in `.memory/decisions/`, one file per decision — this mirrors the classic ADR convention deliberately, since it's a well-understood, portable format any developer or agent recognizes without needing this system's specific documentation to interpret it.

## When a decision gets revisited or reversed

Don't edit the original record to erase what it used to say. Add a new decision record that explicitly supersedes the old one, with a note in the old record pointing to the new one (`Status: Superseded by 0018`). The history of *changing your mind* about something is itself valuable memory — silently rewriting `0012` to reflect a later reversal destroys exactly the kind of context this whole system exists to preserve.

## Feeding the semantic layer

When a decision settles into something that's now just "how this project works" rather than an active choice being defended, its conclusion (not its full reasoning) should also be reflected in `.memory/semantic/architecture.md` — see `memory-consolidation` for when and how that promotion happens.
