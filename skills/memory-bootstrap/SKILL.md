---
name: memory-bootstrap
description: Read a project's persistent memory (.memory/BOOTSTRAP.md, HANDOFF.md, and the latest checkpoint) at the very start of any session, before doing anything else — this is what makes a new agent/harness/provider pick up exactly where the last one left off. Use automatically at the start of every session in a project that has a .memory/ directory, or when a user says "continue where we left off," "pick up from before," or switches from a different AI tool mid-project.
---

# Memory Bootstrap

**This is the single skill every other skill in this system depends on.** If a session doesn't start by reading project memory, nothing downstream (recall, checkpointing, handoff) has anything to build on — this is the entry point of the whole architecture described in [ARCHITECTURE.md](../../ARCHITECTURE.md) of the `agent-memory-protocol` repo.

## What to read, in order, at session start

1. **`.memory/BOOTSTRAP.md`** — the compact (<150 line) orientation: what this project is, current architecture summary, active conventions, non-negotiables. This is the equivalent of a coding agent's `AGENTS.md`/`CLAUDE.md`, scoped specifically to memory-system orientation.
2. **`.memory/HANDOFF.md`** — the most recent explicit handoff note, if one exists (written by `agent-handoff` when a prior session deliberately switched tools/agents).
3. **The most recent file in `.memory/checkpoints/`** — sorted by timestamp, this is the actual last-known state, which matters most specifically because a session might have ended *without* a clean handoff (credit exhaustion, crash) — see `session-checkpoint`'s rationale for why this file, not `HANDOFF.md`, is the authoritative "where things actually stood."
4. **Any `.memory/tasks/*.md` files marked active** — current in-flight work, so the new session doesn't have to be told what's being worked on.

## What this produces

A working understanding of: what the project is, what was being worked on, what decisions have already been made (don't re-litigate them without reason — check `.memory/decisions/` before proposing something that contradicts an existing decision), and what the immediate next step was supposed to be.

## The failure mode this prevents

A new agent starting cold — asking the user to re-explain the project, re-stating already-made decisions as open questions, or worse, silently contradicting a documented decision because it was never read. This is exactly the "no manual reconstruction, no rebuilding context" requirement — it only works if this skill actually fires automatically and actually reads the files, not if it's available but skipped.

## Practical trigger discipline

This should fire **before** the first substantive response in any session where `.memory/` exists in the project root — not after the user asks for it. If `.memory/` doesn't exist yet, this is also the moment to note that and suggest initializing it (see the `agent-memory-protocol` repo's setup instructions), rather than silently proceeding without any memory system in place.

## What NOT to do

Don't re-read the entire episode history (`.memory/episodes/`) at bootstrap — that's expensive and mostly unnecessary; `memory-recall` handles targeted retrieval of specific past episodes when something in the current task actually calls for it. Bootstrap is a fast, cheap orientation pass, not an exhaustive history read.
