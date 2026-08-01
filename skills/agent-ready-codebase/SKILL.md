---
name: agent-ready-codebase
description: Prepare a codebase so agentic loops and graphs can actually operate on it effectively — legible (agents can find what needs changing), executable (work starts at near-zero token cost), and verifiable (outcomes can be proven, not just claimed). Use when setting up a new project for heavy agentic development, when agents repeatedly waste turns figuring out project structure, or when reviewing why an agent's self-reported success doesn't match actual outcomes.
---

# Agent-Ready Codebase

**Source**: AI Builder Club, "Loop Engineering Guide (2026)," which frames this as the most frequently overlooked of the "four ingredients for compounding loops" (attributed to AI Jason's "Loop Engineer: Systemization and Artifacts" framework), despite determining whether autonomous agent operation is feasible at all.

## The three properties

**Legible — agents can locate what needs changing.**
- Maintain a compact (~100-line) index document (`AGENTS.md` or `CLAUDE.md`) that orients an agent quickly, rather than requiring it to explore the whole tree every session.
- Implement custom lints that warn against discouraged patterns specific to the codebase — encoding tribal knowledge as a checkable rule instead of leaving it undocumented.

**Executable — work begins at near-zero token cost.**
- Pre-boot development servers in setup scripts, so an agent doesn't spend its first several turns just getting the environment running.
- Structure the repo to be worktree-friendly, so multiple agents can work in parallel without stepping on each other's uncommitted state.
- Provide scenario-jumping scripts for rapid state transitions — a way to get straight to the relevant application state instead of manually navigating there every time.

**Verifiable — tools exist to prove outcomes, not just claim them.**
- Browser automation with recording (e.g., Playwright with video capture) so a UI-affecting change has actual evidence, not a self-report.
- End-to-end tests that protect core flows, giving any loop or graph node a real thing to check against.
- Avoid designing around agent self-verification patterns — this connects directly to `loop-verifier-design`'s "don't let an agent self-verify" principle, applied at the infrastructure level: build the codebase so *external* verification is cheap and available, not so the agent's only option is judging its own work.

## Why this determines feasibility, not just convenience

A codebase missing these properties doesn't just make agentic work slower — it removes the option of trustworthy unsupervised operation entirely. Per `agentic-loop-autonomy-ladder`, climbing to goal-based or proactive autonomy requires a real verifier; a codebase with no end-to-end tests and no way to record/prove UI outcomes leaves "real verifier" as an unavailable choice, forcing every loop back down to self-verification (weak) or constant human supervision (not actually autonomous) regardless of how well the loop or graph itself is designed.

## Practical checklist

- [ ] An `AGENTS.md`/`CLAUDE.md` index exists and stays close to ~100 lines — a map, not an exhaustive manual
- [ ] A fresh agent session can get the dev environment running without manual setup steps
- [ ] The repo structure tolerates multiple parallel worktrees without state collisions
- [ ] There's at least one way to capture *proof* of a UI or behavioral outcome (recorded video, a passing E2E test, a structured log) beyond an agent's own summary of what it did
- [ ] Custom lints exist for the codebase's specific known footguns, not just generic style rules

## Where this fits in the bigger picture

This is infrastructure work that pays for every loop and graph built afterward — it's the reason `loop-verifier-design`'s "use rule-based checks where possible" tier is even available as an option, rather than falling back to weaker verification because nothing exists to check against.
