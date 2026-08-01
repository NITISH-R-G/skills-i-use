# Skills I Use

> A curated, categorized collection of [Agent Skills](https://agentskills.io) — portable `SKILL.md` capabilities that work across Claude Code, Cursor, Codex, Antigravity, Gemini CLI, and 15+ other coding agents.

This is a personal, working collection — not a framework, not a product. Every skill here is a plain folder with a `SKILL.md` file. Drop the ones you want into `.claude/skills/`, `.agents/skills/`, or your agent's equivalent, and they just work — most trigger automatically, no slash command required.

**⚠️ Read [LICENSE-AND-ATTRIBUTION.md](./LICENSE-AND-ATTRIBUTION.md) before reusing anything.** This repo bundles work from ~30 different authors under their own individual licenses (or no stated license at all). Nothing here is republished as if it were mine unless explicitly marked "original."

---

## Why this exists

Coding agents are only as good as what you hand them. A skill is a small, reusable unit of expertise — "how to write a good test double," "how to design a REST API," "how to run a blameless postmortem" — that an agent reads once, at the exact moment it's relevant, instead of you re-explaining it every session. This repo is what happens when you take that idea seriously and actually build out the collection instead of reinventing the same guidance every project.

## How to use this repo

**Grab everything:**
```bash
git clone https://github.com/<your-username>/skills-i-use.git
cp -r skills-i-use/skills/* your-project/.claude/skills/
```

**Grab one skill:**
```bash
cp -r skills-i-use/skills/tdd your-project/.claude/skills/
```

**Or install skills at the source**, using the [skills.sh](https://skills.sh) CLI, which several of these were originally installed from:
```bash
npx skills add <owner>/<repo>
```
(See [LICENSE-AND-ATTRIBUTION.md](./LICENSE-AND-ATTRIBUTION.md) for the exact source repo of every skill.)

Once installed, most skills are **model-invoked** — your agent reaches for them automatically when a task fits, with zero manual triggering. A handful are **user-invoked** by design (things like `/grill-with-docs`, `/setup-matt-pocock-skills`) because their authors intentionally scoped them to explicit invocation — see each skill's own `SKILL.md` for details.

---

## Categories

### 🧭 Engineering Process — idea to shipped code
The main flow: grill → spec → tickets → implement → TDD → review.

`grill-with-docs` · `to-spec` · `to-tickets` · `implement` · `tdd` · `code-review` · `wayfinder` · `prototype` · `research` · `improve-codebase-architecture` · `diagnosing-bugs` · `triage` · `domain-modeling` · `codebase-design` · `grilling` · `grill-me` · `ask-matt` · `setup-matt-pocock-skills`

### 📚 Software Engineering Reference
Original, citation-backed reference material — design patterns, architecture styles, testing theory, DevOps practice.

`design-patterns` · `software-architecture-styles` · `requirements-engineering` · `refactoring-catalog` · `agile-processes` · `uml-modeling` · `devops-practices` · `testing-strategy` · `test-doubles` · `static-analysis` · `build-systems-and-dependencies` · `documentation-practices` · `engineering-culture` · `observability-and-monitoring` · `microservices-patterns-and-antipatterns` · `twelve-factor-apps` · `container-infrastructure` · `api-design` · `ml-model-evaluation` · `release-engineering-and-incident-response` · `web-performance` · `devsecops` · `minimal-agent-design`

### 🎨 Design & Frontend
`frontend-design` · `ui-ux-pro-max` · `web-design-guidelines` · `writing-guidelines` · `vercel-composition-patterns` · `vercel-react-best-practices` · `vercel-react-native-skills` · `vercel-react-view-transitions` · `theme-factory` · `baseline-ui` · `frontend-slides` · `shadcn-ui` · `enhance-prompt` · `taste-design` and the Stitch design-system family

### 📄 Document & Media Generation
`docx` · `pdf` · `pptx` · `xlsx` · `canvas-design` · `algorithmic-art` · `slack-gif-creator` · `web-artifacts-builder` · `presentation-creator` · MarkItDown MCP

### 🧪 Testing & Quality
`tdd` · `test-doubles` · `testing-strategy` · `static-analysis` · `webapp-testing` · `qa` · `code-review` (×2 variants) · `smoke-check`-style skills from the game-dev bundle

### 🚀 DevOps & Infrastructure
`devops-practices` · `devsecops` · `container-infrastructure` · `twelve-factor-apps` · `git-guardrails-claude-code` · `setup-pre-commit` · `n8n-workflow-patterns` and the full n8n skill family · `wordpress` operations family

### 🧠 AI/ML Engineering
`ml-model-evaluation` · `minimal-agent-design` · `mcp-builder` · `skill-creator` · `skill-scanner` · `skill-writer` · `prompt-optimizer` · the Orchestra-Research AI research library (fine-tuning, quantization, RL training, distributed training references)

### ✍️ Writing & Communication
`writing-guidelines` · `writing-great-skills` · `writing-plans` · `writing-beats` · `writing-fragments` · `writing-shape` · `internal-comms` · `doc-coauthoring` · `edit-article` · `handoff` · `teach`

### 🧩 Domain-Specific
SwiftUI, Angular, materials simulation, iOS Simulator automation, three.js, pixel art/game dev, WordPress plugin development, Sanity CMS, SEO/AEO — see the [full skill index](./skills/) for the complete list.

### 🃏 Novelty / Productivity
`caveman` (ultra-compressed communication mode) and its sub-skills — cuts output tokens ~65% while keeping technical accuracy.

### 🏆 HackerRank Orchestrate Intelligence Layer
**21 skills**, evidence-cited line by line — grounded in HackerRank's published blog posts, the *official public starter repositories* for the May (support-agent) and June (multi-modal-review) events, a direct organizer advice post, an independent engineering analysis (*The Engineer's Notebook*), and a first-hand #1-ranked participant case study. Every claim is tagged `[evidence]` (a direct quote) or `[inference]` (a reasoned extension), so you always know which is which. Full research writeup: **[ORCHESTRATE.md](./ORCHESTRATE.md)** · Self-scoring rubric: **[SCORING-HEURISTIC.md](./SCORING-HEURISTIC.md)** · **[FAQ](./FAQ.md)**

**Core flow:** `orchestrate-phase-gates` · `orchestrate-agent-architecture` · `orchestrate-robustness` · `orchestrate-justification-quality` · `orchestrate-ai-collaboration-transcript` · `orchestrate-self-scoring` · `orchestrate-interview-readiness` · `orchestrate-submission-review`

**Tactical (from the official starter repos):** `orchestrate-schema-guardrails` · `orchestrate-failure-handling` · `orchestrate-naming-and-structure` · `orchestrate-secrets-and-determinism` · `orchestrate-edge-case-testing` · `orchestrate-prompt-engineering` · `orchestrate-multi-strategy-evaluation` · `orchestrate-cost-and-ops-metrics` · `orchestrate-multimodal-evidence-grounding` · `orchestrate-escalation-design`

**Design & resilience (from an independent analysis + a #1-ranked case study):** `orchestrate-input-tracing` · `orchestrate-input-validation-and-overrides` · `orchestrate-checkpoint-resilience`

Also maintained as a standalone, Orchestrate-focused repo: **[hackerrank-orchestrate-skills](https://github.com/NITISH-R-G/hackerrank-orchestrate-skills)**.

### 🔀 Loop & Graph Engineering
**8 skills** on designing agentic loops (verifier-as-bottleneck, stop conditions, the turn→goal→time→proactive autonomy ladder) and multi-agent graphs (nodes, edges, fan-out/fan-in, a real reviewer node) — sourced from AI Builder Club's 2026 guides and cross-checked against academic literature (ReAct, Reflexion, and a 2026 arXiv paper on verified multi-agent orchestration that independently corroborates the core "decoupled verification" claim). Every claim is evidence-tiered: practitioner synthesis vs. academic corroboration vs. inference. Full sourcing: standalone repo's [RESEARCH.md](https://github.com/NITISH-R-G/graph-engineering-skills/blob/master/RESEARCH.md).

`loop-vs-graph-decision` · `loop-verifier-design` · `loop-stop-conditions` · `agentic-loop-autonomy-ladder` · `graph-node-and-edge-design` · `graph-reviewer-node` · `agent-ready-codebase` · `compounding-loop-architecture`

### 🧠 Persistent Cross-Agent Memory
**9 skills** implementing a portable, git-committed memory system so a project's context survives switching AI models, providers, coding agents, IDEs, or harnesses mid-work — including abrupt cutoffs (credit exhaustion, crashes), not just clean handoffs. Built on git-committed files plus the `AGENTS.md`/`CLAUDE.md` bootstrap convention every major coding harness already reads, with the official Anthropic MCP memory server adopted for structured recall (Mem0/Redis documented as opt-in upgrades, not rebuilt). Sourced from LangMem, LangChain's long-term memory docs, a 5-pattern architecture catalog, Redis's memory architecture guide, and academic literature — full research report, gap analysis, and architecture: standalone repo's [RESEARCH.md](https://github.com/NITISH-R-G/agent-memory-protocol/blob/master/RESEARCH.md) / [ARCHITECTURE.md](https://github.com/NITISH-R-G/agent-memory-protocol/blob/master/ARCHITECTURE.md).

`memory-bootstrap` · `memory-capture` · `session-checkpoint` · `agent-handoff` · `memory-recall` · `decision-log` · `memory-consolidation` · `memory-hygiene` · `repo-awareness`

Note: `agent-handoff` is a deliberate, richer handoff note distinct from mattpocock's already-installed `handoff`/`claude-handoff` skills (which cover a different kind of session-transfer) — see the skill's own description for how they differ.

Also maintained as a standalone repo, with the reusable `.memory/` template and MCP setup: **[agent-memory-protocol](https://github.com/NITISH-R-G/agent-memory-protocol)**.

Also maintained as a standalone repo: **[graph-engineering-skills](https://github.com/NITISH-R-G/graph-engineering-skills)**.

---

## What makes this collection different

- **Cross-agent by construction.** Every skill uses the open `SKILL.md` standard — the same files work in Claude Code, Cursor, Codex, Antigravity, and more, no adaptation needed.
- **Curated, not dumped.** Every skill in here was individually reviewed before inclusion — repo existence verified, publisher checked, permissions/dependencies read, duplicates and abandoned projects filtered out. (Full review methodology and findings: [REVIEW-NOTES.md](./REVIEW-NOTES.md).)
- **Honest about provenance.** Every skill is traceable to its original source repo and author. See [LICENSE-AND-ATTRIBUTION.md](./LICENSE-AND-ATTRIBUTION.md).
- **A meaningful chunk is original.** 23 of these skills were written from scratch — reference material on software architecture, testing theory, and engineering practice, citing (never copying) established texts like *Software Engineering: A Modern Approach* and *Software Engineering at Google*.

## Contributing

Found a skill that should be here? Open a PR adding it to `skills/`, with its source repo and license noted in `LICENSE-AND-ATTRIBUTION.md`. Skills without a clear, verifiable source won't be merged.

## License

The curation, this README, and the 23 skills marked "original" in [LICENSE-AND-ATTRIBUTION.md](./LICENSE-AND-ATTRIBUTION.md) are MIT licensed (see [LICENSE](./LICENSE)). Every other skill retains its original author's license — this repo does not relicense third-party work. See [LICENSE-AND-ATTRIBUTION.md](./LICENSE-AND-ATTRIBUTION.md) for the full breakdown.
