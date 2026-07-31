---
name: devops-practices
description: Reference for version control branching strategies and continuous integration/continuous deployment (CI/CD) practices — trunk-based development vs. GitFlow, what a good CI pipeline checks and in what order, and the distinction between continuous delivery and continuous deployment. Use this whenever the user is setting up a branching strategy, designing or debugging a CI pipeline, deciding on a release process, or asking how often to merge, branch, or deploy.
---

# DevOps Practices

The throughline across branching strategy and CI/CD is the same idea from different angles: the longer code stays unintegrated and unverified, the more expensive integration becomes. Every practice below is either shortening that gap or catching problems before they compound.

## Version control: branching strategies

### Trunk-Based Development

**Shape**: everyone commits directly to `main` (trunk), or through very short-lived branches (hours, not days) that merge back quickly. Incomplete features ship behind feature flags rather than staying on a long-lived branch until "done."

**What it buys you**: integration happens continuously, in small increments — the failure mode of a long-lived branch (weeks of divergence, then a massive, conflict-ridden, hard-to-review merge) structurally can't happen, because branches never live long enough to diverge that far. This is XP's "continuous integration" practice, made concrete as a branching policy.

**What it requires to work safely**: a CI pipeline good enough to catch regressions on every merge to trunk (see below) — trunk-based development without solid CI means broken code lands on `main` and every teammate pulls it. Feature flags for incomplete work, so half-built features can merge to trunk without being user-visible yet.

**Fits**: teams with strong CI discipline and a codebase where features can be decomposed into small, mergeable increments (which, not coincidentally, is also what makes small-ticket, tracer-bullet-style planning work — see `to-tickets`).

### GitFlow (and similar long-lived-branch strategies)

**Shape**: long-lived `develop` and `main` branches, feature branches cut from `develop` and merged back when complete, release branches for stabilization, hotfix branches for urgent production fixes.

**What it buys you**: a clear, structured process for coordinating releases with defined stabilization windows — useful when releases are scheduled, versioned events (e.g., shipping a specific version number to enterprise customers on a fixed cadence) rather than continuous rolling deploys.

**What it costs**: feature branches that live for days or weeks accumulate drift from `develop`, and the eventual merge is where that drift resolves all at once — the exact integration pain trunk-based development is designed to avoid. Also more branches to keep track of, more merge ceremony, and (this is the real cost) a longer average time between a line of code being written and it being verified against everyone else's latest work.

**Fits**: software with genuinely versioned, scheduled releases (desktop software, embedded firmware, anything with a formal release/certification process) where a stabilization branch model matches how the business actually ships.

### The general trend and how to pick

The industry has moved toward trunk-based development for continuously-deployed software (web services, SaaS) because deploy frequency there is high enough that "wait for a release branch to stabilize" doesn't match how the software actually ships. GitFlow-style long-lived branches still make sense specifically where releases are discrete, versioned events. The question to ask isn't "which is more modern" — it's "does this software ship continuously or in discrete versioned releases," and let that answer decide.

**Branch protection basics regardless of strategy**: require passing CI before merge to any shared/long-lived branch, require review (see `code-review`) before merge, and never allow force-push to a shared branch — these are cheap, universally-applicable safety nets independent of which branching strategy is in use.

## Continuous Integration (CI)

**What it actually is**: automatically building and testing every change, on every push/merge, without a human triggering it manually. The point is fast, automatic feedback — a broken change is caught within minutes, not discovered days later when someone else's work collides with it.

**A well-ordered pipeline runs cheap, fast checks before expensive, slow ones** — fail fast, so a trivial mistake (a lint error, a type error) doesn't wait behind a 20-minute integration test suite to be reported:

1. **Lint/format check** — seconds. Catches style violations and some classes of bugs (unused variables, unreachable code) before anything else runs.
2. **Type check** — seconds to low minutes. Catches a large class of bugs statically, before any test needs to run to find them.
3. **Unit tests** — should be fast (seconds to a couple minutes for a healthy suite) — these are the bulk of `tdd`'s red-green loop, and slow unit tests erode the tight feedback loop that makes TDD viable in the first place.
4. **Build** — compiles/bundles the actual deployable artifact.
5. **Integration/E2E tests** — slower, exercise real boundaries (database, network, browser). Run after the faster checks specifically so a failure here doesn't also mean waiting through a lint failure that a 2-second check would have caught immediately.

**What "green CI" actually promises, and what it doesn't**: a passing pipeline means the asserted properties hold — the things the tests actually check. It does not mean the code is correct in every sense; it means it's correct in every sense someone thought to write a check for. This is the same caveat covered in the `tdd` skill's discussion of automated checks — worth remembering before treating "CI is green" as equivalent to "this change is definitely fine."

**Flaky tests are a CI emergency, not a nuisance to tolerate.** A test that fails intermittently for reasons unrelated to the code under test teaches the team to re-run failures without investigating — which means the next *real* failure, the one that actually matters, gets re-run and ignored too. Fix or quarantine flaky tests immediately rather than letting them accumulate; a CI suite where "just re-run it" is normal has already lost the thing CI exists to provide (trustworthy, fast feedback).

## Continuous Delivery vs. Continuous Deployment

These two terms get used interchangeably and shouldn't be — the difference is a single, meaningful gate:

| | Continuous Delivery | Continuous Deployment |
|---|---|---|
| Every merge to trunk that passes CI is... | Automatically built into a release-ready artifact | Automatically deployed to production |
| Final step to production | Manual trigger (a person clicks "deploy") | Fully automatic, no human step |
| What it requires | A pipeline that reliably produces a deployable artifact | Everything Delivery requires, *plus* enough confidence in the automated checks that no human review step is needed before production traffic sees the change |

**Continuous Delivery** is the more common target: the team could deploy any passing build to production at any time — the artifact is always release-ready — but a human decides when that actually happens (batching for a release window, coordinating with support, etc.).

**Continuous Deployment** removes that last human gate entirely. This is a much higher bar — it requires enough trust in the automated pipeline (comprehensive test coverage, feature flags to de-risk incomplete work, fast rollback/monitoring to catch what tests miss) that shipping straight to production with no human in the loop is genuinely safe, not just fast. Don't reach for Continuous Deployment as a default goal — it's earned by a mature pipeline, not a starting configuration.

**Rollback strategy matters more than deploy speed.** However fast a deploy pipeline is, something will eventually reach production broken — a fast, well-tested pipeline reduces how often, not to zero. The real question worth asking before optimizing deploy speed further: how fast can a bad deploy be detected and reverted? Feature flags (toggle off without a redeploy), canary/staged rollouts (catch a bad deploy on 5% of traffic before it reaches 100%), and a fast, well-rehearsed rollback path are what make continuous deployment survivable in practice.

## How this connects to the rest of the toolkit

Trunk-based development plus solid CI is the infrastructure that makes `implement`'s tight TDD loop and `to-tickets`'s vertical-slice tickets actually pay off end-to-end — small, frequently-integrated changes are exactly what a tracer-bullet ticket produces, and they're exactly what trunk-based development wants merged. A team running GitFlow with long-lived feature branches will find vertical-slice tickets harder to land continuously, since each one still waits behind the feature branch's own merge cadence.
