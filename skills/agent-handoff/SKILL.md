---
name: agent-handoff
description: Write an explicit handoff note to .memory/HANDOFF.md when consciously ending a session to switch AI tools, models, providers, or harnesses — a deliberate, richer counterpart to the continuous safety net session-checkpoint provides. Use when the user says they're switching to a different agent/tool/provider, when ending a session that isn't finished, or when explicitly asked to prepare a handoff.
---

# Agent Handoff

**Distinct from `session-checkpoint`**: a checkpoint is a cheap, frequent, continuous safety net written throughout a session. A handoff is a deliberate, one-time, richer note written specifically because you *know* the next session will be a different tool, model, or provider picking this up cold — it can afford more context than a checkpoint, because it's written once, at a known transition point, not on a tight cadence.

## What makes a handoff different from just "a bigger checkpoint"

A handoff should anticipate that the next agent may have **zero shared context** with this session — not just a different tool, potentially a different underlying model with different tendencies, different tool-calling conventions, different strengths. Write it assuming the reader knows nothing about this specific conversation, only what's in `.memory/` generally.

## Handoff note structure

```markdown
# Handoff: 2026-08-01, ending session with Claude Code (credits exhausted mid-task)

## What this session accomplished
- Extracted TokenService from auth middleware (decisions/0012)
- Wrote unit tests for TokenService (all passing)
- Started but did NOT finish: migrating callers of the old inline refresh logic

## Exact current state
3 of 7 call sites migrated to TokenService. Remaining: UserController, AdminController,
the background refresh cron job. Migration pattern is in decisions/0012 — follow it exactly,
don't improvise a variant.

## Known open questions (not yet decided)
Whether TokenService should own the refresh interval config or receive it injected — see
checkpoints/2026-08-01T14-30-00.md for the tradeoff notes. Needs a decision before the cron
job migration specifically, since that's where the interval actually matters.

## Do NOT
- Don't re-litigate the extract-vs-inline decision (decisions/0012) without new evidence —
  this was already tried both ways and extraction won.
- Don't assume the 3 migrated call sites are done-done — they're migrated but not yet
  covered by integration tests, only the unit tests on TokenService itself.

## Suggested next step
Migrate UserController next (simplest of the remaining 3), then resolve the config-vs-injected
question before touching the cron job.
```

## Why "Do NOT" is its own section, deliberately

The single most expensive failure mode in a cross-agent handoff isn't lost progress — it's a new agent **re-deciding something already decided**, burning time and potentially reversing a considered tradeoff because it wasn't told the alternative had already been tried and rejected. An explicit "don't re-litigate X" list, backed by a pointer to the actual decision record, is cheap to write and directly prevents this.

## Triggering this automatically

Per the request's own goal ("the user should never need to remember... write transition prompts"), this should fire proactively when the conversation signals an intentional end-of-session-for-a-switch — the user saying "I'm out of credits," "switching to Codex," "let me try this in Cursor," or similar — not only when explicitly asked. If a session simply stops without any such signal, `session-checkpoint`'s continuous writes are what the next session falls back on instead — this is why both skills exist rather than relying on handoff alone.

## Where the new agent finds this

`memory-bootstrap` reads `HANDOFF.md` as step 2, right after the general project orientation — a fresh agent in a new tool sees this before doing anything else.
