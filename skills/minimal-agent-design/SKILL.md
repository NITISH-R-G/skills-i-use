---
name: minimal-agent-design
description: Reference for designing minimal, hackable coding-agent harnesses — linear message history over branching state, bash-only tool surfaces over bespoke tool abstractions, and subprocess isolation for safe sandboxing — inspired by mini-swe-agent's ~100-line agent achieving strong SWE-bench results through radical simplicity. Use this whenever the user is building their own agent harness or orchestration loop, debugging why an agent's behavior is hard to trace, deciding whether to add a custom tool or just let the model use a shell, or evaluating whether an agent's architecture has grown more complex than the problem actually requires.
---

# Minimal Agent Design

The provocative question worth sitting with before building any agent harness: what's the simplest thing that could possibly work, and how much of the complexity in a typical agent scaffold is actually load-bearing versus accumulated by default? mini-swe-agent — a ~100-line agent scoring over 74% on SWE-bench Verified — is existence proof that a large fraction of typical agent-scaffold complexity is optional, not load-bearing. That doesn't mean every agent should be 100 lines; it means complexity should be added because a specific problem demands it, not by default.

## Linear message history over branching state

**The design choice**: every step appends to one flat, ordered message log — no branching decision trees, no separate state object tracked alongside the conversation, no divergence between "what the model was told happened" and "what actually happened."

**Why this beats stateful/branching alternatives for most agent work**:
- **What you see is what the model saw.** The entire prompt at any step is reconstructable by reading the log up to that point — there's no hidden state influencing the model's next action that isn't visible in the transcript. This is the single biggest lever for debugging: when an agent does something unexpected, the linear log tells you exactly what it was looking at when it decided to do that, with nothing to reconstruct or infer.
- **No state-sync bugs.** A separate state object (tracked outside the message history, injected back in at each step) creates a whole class of bug where the state object and the message history disagree — the model "remembers" something the state object has since changed, or the state object has information the model was never actually shown. A linear history structurally can't have this bug, because there's only one source of truth.
- **Trajectories are directly useful data.** A linear history *is* the exact input/output pairs a fine-tuning process would want — no extraction or reconstruction needed from a more complex internal representation. This is a concrete, practical payoff of the design choice, not just an aesthetic preference for simplicity.

**When you'd actually need branching/stateful history instead**: genuine multi-path exploration where several alternatives need to be tried and compared *within one agent's working memory* (not sequentially, but structurally in parallel) — that's a real requirement complex state can address. But this is rarer than typical agent scaffolds assume; most "I need branching state" instincts are actually solvable with a linear history plus periodic summarization/compaction (see the `handoff` and memory-related skills already in this toolkit) rather than genuine in-flight branching.

## Bash-only tools over bespoke tool abstractions

**The design choice**: instead of building a `read_file` tool, a `write_file` tool, a `run_tests` tool, a `git_diff` tool, and so on — give the model exactly one capability, a shell, and trust it to compose `cat`, `sed`, `pytest`, `git diff` itself, the way a human engineer would.

**Why this is a real trade-off, not just laziness**:
- **Model-centric, not scaffold-centric.** Building ten bespoke tools is ten places where the tool's design might not match how the model naturally wants to express an action — a mismatch that shows up as the model fighting the tool's exact parameter shape instead of just doing the thing. A shell has no such mismatch, because it's the same interface a human uses and the model has enormous amounts of training data on using it well.
- **One tool surface, universal compatibility.** A bespoke tool-calling interface ties the agent to whichever models support that specific tool-calling API cleanly — a bash-only agent works with any model that can produce text, including models with weaker or no native tool-calling support, since "the tool" is just "write a shell command."
- **Composability for free.** A model that can use bash can chain `grep | sed | sort` in one line — expressing an operation no bespoke tool anticipated, without the harness author having had to predict and build for that specific combination in advance. A fixed tool set can only do what it was explicitly built to do; a shell can do anything the model can express as a shell pipeline.

**The real cost, worth naming honestly**: a bash-only agent has less structured guardrails than a bespoke tool with validated parameters — a bespoke `write_file(path, content)` tool can reject an obviously-wrong path before anything happens; raw bash trusts the model's command to be well-formed and lets failures surface as shell errors instead of being caught earlier. This matters more for permission/safety boundaries (see below) than for correctness — a well-designed sandbox contains the blast radius of a bad bash command regardless of how well-formed it was.

**When bespoke tools earn their complexity anyway**: a capability genuinely not expressible through shell composition (calling a proprietary internal API with a complex auth flow, for instance) needs a real tool. And a tool that adds a genuine safety boundary bash alone can't provide (a `run_tests` tool that enforces a timeout and resource limit no ad hoc bash invocation would reliably include) is adding real value, not just structure for its own sake. The discipline is asking, for each proposed tool, "does this let the model do something it genuinely couldn't compose from bash, or is this convenience/guardrail wrapping around something bash already does" — the latter is often not worth the added scaffold complexity.

## Subprocess isolation for safe, swappable sandboxing

**The design choice**: each action executes as an independent subprocess call (`subprocess.run`), not as a command sent into one long-lived, persistent shell session that accumulates state (environment variables, working directory changes, background processes) across the whole agent run.

**Why this matters for stability, not just cleanliness**: a persistent shell session accumulates invisible state over a long agent run — a `cd` three steps ago silently changes what a later relative-path command actually does, an environment variable set once affects everything after it in ways that aren't visible in the linear history unless you were tracking shell state separately (reintroducing exactly the state-sync problem linear history was trying to avoid). Independent subprocess calls mean each action's effective environment is fully determined by what's explicit in that action, not by an invisible accumulation of everything before it.

**What this buys for sandboxing specifically**: because each action is just "run this subprocess," swapping the execution backend (local process → `docker exec` → a sandboxed runtime) is a change to *one place* in the harness — the subprocess-execution layer — with zero change needed to the agent's decision-making logic. A harness built around a persistent, stateful session is much harder to sandbox cleanly, because the sandbox has to somehow preserve that session's accumulated state across the boundary, which is a much harder problem than isolating one action at a time.

**Direct connection to this session's own vocabulary**: this is the same underlying idea as the `sandbox` concept already covered elsewhere in this toolkit's AI-coding-dictionary knowledge — isolating blast radius by construction, rather than trying to police a shared, stateful environment after the fact.

## "Hackability over complexity" as a standing design filter

The unifying principle behind all three choices above, worth applying as an explicit filter on any addition to an agent harness: **understandable beats sophisticated**. A ~100-line agent class that someone can read start to finish and modify with confidence is worth more, for most practitioner use cases, than a more "capable"-looking agent whose behavior on any given step requires tracing through several layers of abstraction to predict. This isn't an argument that sophistication is never warranted — a research agent exploring genuinely novel scaffolding techniques has different goals than a practitioner's daily-driver tool — but it's a useful default to check any added complexity against: does this abstraction layer earn its cost in genuinely new capability, or is it complexity that accumulated because it seemed like the "proper" way to build an agent, without a specific problem actually requiring it?

**A concrete question to ask before adding anything to an agent harness**: if this piece were removed, what specific failure would result? If the honest answer is "nothing observable, it's just architecturally tidier," that's a signal the addition isn't paying for itself — the same test the `writing-great-skills` skill in this toolkit applies to skill content (the "no-op test") applies just as well to agent-scaffold design.

## Practical checklist when designing or reviewing an agent harness

- Is there a single source of truth for what the model has seen, or does state live partly in a separate structure that could drift from the model's actual context?
- For each bespoke tool in the harness, could the model accomplish the same thing by composing a shell command instead — and if so, what does the bespoke tool actually add?
- Does each agent action execute in an isolated context, or does state silently accumulate across actions in a way that isn't visible in the transcript?
- Could the execution/sandboxing backend be swapped (local → containerized → remote) by changing one layer, or would that require touching the agent's core decision logic?
- For any abstraction under consideration: what specific failure would occur if it were removed? If none, it's a candidate for cutting.
