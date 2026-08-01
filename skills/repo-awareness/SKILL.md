---
name: repo-awareness
description: Feed repository state (recent commits, active branch, open TODOs, recently modified files) into project memory so a new session/agent understands current repo context without re-deriving it from scratch. Use at session start alongside memory-bootstrap, or when project memory's sense of "current state" seems stale relative to the actual repo.
---

# Repo Awareness

**Scope note**: this is deliberately narrow — feeding *repository-derived* signals into the memory system, not a general codebase-analysis or architecture-review skill. For deeper codebase quality/architecture review, this ecosystem already has `improve-codebase-architecture` and `codebase-design` — reach for those directly rather than duplicating their job here. This skill's only concern is: what does the repo's current state tell a memory-aware session that it should know.

## What to check, cheaply

- **Recent commits** (`git log --oneline -20` or similar) — what's actually changed recently, which may not match what `.memory/` says was last worked on if changes happened outside a memory-aware session (a different tool, a human directly)
- **Current branch and its relationship to main** — whether there's unmerged work in flight
- **Files modified but not committed** — uncommitted work in progress that a fresh session needs to know about before touching those files
- **TODO/FIXME comments in recently touched files** — lightweight, in-code signals that may not have made it into `.memory/tasks/` yet
- **Open items in the project's issue tracker, if the project has one integrated** (see the `mattpocock`-style `issue-tracker` conventions elsewhere in this collection if the project uses local markdown issue tracking, or a real GitHub Issues/Linear integration if configured)

## Reconciling repo state with memory state

The interesting case is **divergence** — when the repo shows activity that `.memory/` doesn't reflect (commits from a session that didn't use this memory system, or from a human working directly) or vice versa (memory describes in-flight work that the repo shows as already merged and done). Flag this explicitly rather than silently trusting one source over the other:

*"`.memory/tasks/token-service-migration.md` says 3 of 7 call sites are migrated, but `git log` shows a commit 'migrate remaining TokenService call sites' — checking which is accurate before continuing."*

This reconciliation is exactly what prevents the memory system from becoming a second, stale source of truth that actively misleads once real repo activity happens outside its awareness — a real risk for any external memory layer, named implicitly by every source's emphasis on validation and forgetting.

## Feeding this back into memory

Findings from a repo-awareness pass update `.memory/tasks/*.md` files to match reality (not the other way around — the repo, specifically committed history, is the more authoritative source when the two disagree about *what actually happened*, even though `.memory/` may be more authoritative about *why* it happened). Genuinely new information (a TODO that reveals unplanned work, a branch that reveals abandoned work) gets captured via `memory-capture` like anything else.

## Practical cadence

Run this at session start, alongside `memory-bootstrap` — the two are complementary: bootstrap reads what the memory system *believes*; repo-awareness checks that belief against what the repo *actually shows*, cheaply, before proceeding on potentially stale assumptions.
