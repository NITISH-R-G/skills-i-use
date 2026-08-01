---
name: agentic-loop-autonomy-ladder
description: Climb agentic autonomy one rung at a time — turn-based, then goal-based, then time-based, then proactive — rather than jumping straight to unsupervised scheduled agents, since each rung hands off strictly more control and skipping rungs produces doom loops. Use when deciding how much autonomy to give an agent, when setting up a scheduled/recurring agent task, or when an unsupervised loop keeps retrying the same broken approach.
---

# The Agentic Loop Autonomy Ladder

**Source**: AI Builder Club, "The 4 Types of Agentic Loops" and "Loop Engineering Guide (2026)" — describes Claude Code's own `Auto` mode, `/goal`, `/loop`, and Routines/scheduled-task features as concrete implementations of each rung.

## The four types, by what they hand off

Each type hands off strictly more of the loop than the one before it — the source: *"Each hands off strictly more of the loop than the one before it, which is why they form a ladder rather than a menu."* Not four alternatives to pick between freely; a progression.

1. **Turn-based — hand off tool approval.** The agent approves its own tool calls within one turn without interruption (e.g., Claude Code's Auto mode), but still decides when to *stop* based on its own judgment — the weakest verifier available. Fine for supervised, interactive work; vulnerable to doom-looping once left unattended.
2. **Goal-based — hand off the stop condition.** A measurable completion condition is set explicitly (e.g., Claude Code's `/goal`); after each turn, an evaluator (ideally a small, fast, independent model, per the source) checks whether the condition holds. This is the pivotal rung — the first one where genuinely unsupervised operation becomes defensible, because a real verifier now governs stopping instead of the agent's own judgment.
3. **Time-based — hand off the trigger to a clock.** The loop reruns on an interval (e.g., `/loop`), reacting to elapsed time rather than the agent's own sense of completion. Typically session-scoped — tied to an open session, expiring after some window.
4. **Proactive — hand off the trigger to a schedule or event.** Runs with no open session required at all — scheduled routines, desktop scheduled tasks, or event-driven triggers. The most autonomous rung, and the one where a weak verifier is most expensive to have missed.

## The doom loop: the shared failure mode across all four

*"Hand off the trigger and the stop condition without a real verifier, and you don't get a productive agent; you get one doom-looping"* — retrying the same broken approach, burning budget producing confident garbage, with nobody watching. This risk exists at every rung; it just gets more expensive and less visible as autonomy increases, because turn-based doom-looping is watched by a human in real time and proactive doom-looping isn't watched by anyone until someone checks the bill or the output.

**The prevention rule, stated directly in the source**: *"the loop's stop condition must be a verifier the agent has to satisfy, not a vibe it's allowed to declare."*

## The climbing sequence

1. **Stage 1 — turn-based, supervised.** Run in Auto mode while actually watching, to learn the task's real failure patterns before trusting it unsupervised.
2. **Stage 2 — goal-based.** Translate a vague goal ("improve the auth flow") into a measurable condition ("login test passes, no existing test breaks"). This is where `loop-verifier-design` and `loop-stop-conditions` do their work — the goal-based rung is only safe once that verifier is actually solid.
3. **Stage 3 — time-based.** Once the verifier has proven reliable and the triggers are genuinely external (CI results landing, a deployment completing), move to interval-based polling.
4. **Stage 4 — proactive.** Only after the system has been proven at the previous rungs — promote to scheduled/event-driven operation for true unsupervised autonomy.

**Core principle**: *"you slide toward autonomy only as far as your ability to verify allows."* Skipping rungs — going straight from turn-based to proactive because scheduling is easy to set up — directly produces doom loops, because the verifier that would have made the jump safe was never actually built or tested.

## Practical use

Before configuring any scheduled or event-triggered agent, ask which rung you're actually on. If the honest answer is "I haven't run this supervised yet and I don't have a measurable stop condition," the fix isn't a better schedule — it's going back to Stage 1 and 2 first.
