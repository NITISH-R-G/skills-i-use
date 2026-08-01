---
name: graph-node-and-edge-design
description: Design the nodes, edges, and shared state object for a multi-agent graph — specialized work units, routing between them (straight, conditional, fan-out, fan-in), and an explicit state schema — once loop-vs-graph-decision has confirmed a graph is actually warranted. Use when architecting a multi-agent system, when designing fan-out/fan-in parallelism, or when a graph's "shared state" is vaguely defined instead of an explicit schema.
---

# Graph Engineering: Nodes, Edges, and Shared State

**Source**: AI Builder Club, "Graph Engineering Guide (2026)" — describes the pattern as implemented (with different specifics) by LangGraph's `StateGraph`, Microsoft AutoGen's GraphFlow, and Google ADK's graph-based workflow agents, all predating the "graph engineering" terminology itself.

## The three essential components

1. **Nodes** — work units, typically specialized agents or deterministic functions. Real specialization means a distinct prompt, tool set, and clean context per node — not one agent's transcript switching roles mid-conversation (see `loop-vs-graph-decision`'s 4-question test, question 1).
2. **Edges** — routing between nodes: straight (always proceed to the next node), conditional (branch based on a check), fan-out (spawn multiple parallel branches), fan-in (wait for parallel branches and merge).
3. **Shared state** — an object that travels along the edges, carrying task data, notes, drafts, and verdicts, growing as it passes through nodes.

## The canonical starter pattern

**Researcher → Writer → Reviewer**, with a conditional edge: pass → Ship; fail → loop back to Writer. The state object grows explicitly at each step:
```
{task, notes}                          # after Researcher
{task, notes, draft}                   # after Writer
{task, notes, draft, verdict}          # after Reviewer
```
Note this is small and explicit — every field's origin and purpose is traceable, which is what makes the graph auditable rather than a black box passing an ever-growing, loosely-typed blob between nodes.

## Fan-out/fan-in: the genuine parallelism a loop can't do

This is one of the three concrete capabilities that distinguishes a real graph from a loop (see `loop-vs-graph-decision`): spawn N parallel branches, wait for all (or a defined subset) to complete, then merge results into one continuation. The source's example — a daily-brief system running ten research nodes concurrently across different sources, then consolidating into one document — only works this way in a graph; a loop can only do this sequentially, one source at a time.

## Design the state schema before writing any node

Per the source's own checklist item — "draw edges before coding" — the state schema should be decided explicitly, not discovered by accretion as nodes get written. For each node, ask: what does it need to *read* from the state that upstream nodes produced, and what does it need to *write* for downstream nodes to consume? An undefined schema is exactly what produces the guide's named failure mode: silent state leaks between nodes, where a node quietly depends on a field another node happened to set, with no contract enforcing it.

## Practical checklist

- [ ] Each node has a distinct prompt/tool set — not one agent role-switching within a shared transcript
- [ ] The state object's fields are explicitly listed per stage, not an unstructured "whatever accumulates"
- [ ] Fan-out/fan-in edges are used only where real, useful parallelism exists — not as a default topology
- [ ] Conditional edges (like the loop-back-to-Writer pattern) have an explicit condition tied to a verifier's verdict, not an implicit judgment call buried in a node's prompt
- [ ] The full graph — nodes and edges — can be drawn as a diagram someone unfamiliar with the code could read

## Where this connects

Every node in a graph is, internally, running its own loop (see `loop-verifier-design`, `loop-stop-conditions`) — the graph layer coordinates *between* nodes; it doesn't replace the need for each node to have a real verifier and stop condition of its own. See `graph-reviewer-node` for the specific, high-leverage case of designing the Reviewer node itself.
