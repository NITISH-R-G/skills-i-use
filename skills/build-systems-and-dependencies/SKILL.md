---
name: build-systems-and-dependencies
description: Reference for build system philosophy (task-based vs. artifact-based, module granularity) and dependency management theory (semantic versioning's real limitations, diamond dependencies, the live-at-head alternative). Use this whenever the user is configuring a build system, debugging a dependency conflict, arguing about module boundaries or build file structure, deciding how to version a library, or asking why a dependency upgrade broke something SemVer said it wouldn't.
---

# Build Systems and Dependency Management

Two related but distinct problems: how do you turn source into artifacts correctly and reproducibly (build systems), and how do you consume other people's code without your dependency graph becoming unmanageable (dependency management). Both get harder non-linearly as a codebase grows, which is why the "obvious" approach (a shell script; SemVer and hope) works fine at small scale and fails in specific, predictable ways at large scale.

## Build system philosophy: task-based vs. artifact-based

**Task-based** (Make, most shell-script-driven builds, early Ant/Gradle usage): you write scripts that say "to build X, run these commands, in this order." The build system executes what you told it to.

**Problem at scale**: nothing stops a task from doing something not declared in its inputs/outputs — a task that reads a file it didn't declare as a dependency works fine until that file changes and the build system, having no idea the task depended on it, serves a stale cached result. This class of bug (incorrect incremental builds, becoming worse as the script graph grows) is close to inherent to the task-based model, not just a matter of writing it more carefully — the model has no way to *verify* your declared dependencies match your task's actual behavior.

**Artifact-based** (Bazel, Buck, and similar): you declare *what* each target produces and *what* it depends on (as data, not as executable steps); the build system decides *how* to actually invoke the compiler/linker to satisfy that graph. Because the dependency graph is declarative data rather than imperative script, the build system can verify a target only reads what it declared, cache correctly based on actual inputs, and safely parallelize/distribute work across machines — task-based systems can't safely do any of these without risking a task's undeclared side effects being invisible to the caching layer.

**Practical implication**: task-based systems are simpler to start with and fine for small projects with few contributors and no build farm; artifact-based systems pay off specifically once you need reliable incremental builds, safe parallelization/distribution, or many people modifying the build graph concurrently without silently breaking each other's caching. The crossover point is usually "the build is slow enough or flaky enough that people work around it" — that's the signal it's worth the migration cost.

## Module granularity — the 1:1:1 idea

**The organizing principle**: one build target maps to one directory maps to one logical component (roughly: "one thing, one place, one build rule for it"), kept fine-grained rather than bundling many unrelated things into one large target.

**Why fine-grained targets, not one big one**: a large, coarse-grained build target means any change anywhere in it forces rebuilding/retesting the whole thing, and any consumer depending on any part of it transitively depends (and gets rebuilt when anything changes) even in the parts they don't use. Fine-grained targets let the build system's incrementality actually work — change one small target, only rebuild/retest what depends on that specific target, not everything downstream of the large bundle it used to live inside.

**The cost, and why it's worth it anyway**: many small targets means more build files to write and maintain, and more boilerplate declaring inter-target dependencies. This is a real, ongoing cost — but it's linear and predictable, versus the coarse-grained alternative's cost (slow builds, unnecessary rebuilds, unclear ownership of what's inside a large bundled target) which compounds as the codebase grows and doesn't show up until it's already painful to unwind.

## Minimizing visibility — the other half of good module boundaries

Declaring a target's dependencies is only half of a healthy build graph — the other half is controlling who's *allowed* to depend on a given target (visibility/access control at the build-system level, not just language-level access modifiers). A target with unrestricted visibility gets depended on by things it was never designed to support, and by the time that's discovered, removing the dependency means coordinating a change across everyone who took the shortcut. Default new internal targets to restricted visibility (visible only to their intended consumers) and widen deliberately when a real, legitimate new consumer appears — narrowing visibility *after* the fact requires finding and migrating every unintended consumer, which is the expensive direction to go.

## Dependency management: the diamond dependency problem

**The shape of the problem**: your project depends on library A and library B; A depends on library C version 1; B depends on library C version 2. Which version of C does your build actually use? If A and B's usage of C aren't compatible with a single shared version, there may be no correct answer at all without changing A or B.

**Why this gets worse, not better, as a dependency graph grows**: the number of potential version conflicts grows with the depth and breadth of the transitive dependency graph, and each individual library owner has no visibility into what version constraints every downstream consumer's *other* dependencies require. No single actor in the graph has enough information to prevent the conflict — it's a structural property of many independently-versioned things depending on a shared thing, not a mistake any one party made.

## Why semantic versioning doesn't fully solve this

**What SemVer promises**: `MAJOR.MINOR.PATCH`, where major version bumps signal breaking changes, minor/patch signal backward-compatible changes — in theory letting a consumer safely accept "any minor/patch update" without re-verifying compatibility.

**Where the promise breaks down in practice**:
- **The promise depends entirely on human judgment being correct and consistent** about what counts as "breaking" — a change that the library author considered a safe patch might still break a consumer relying on an implementation detail that was never part of the documented contract (this is the more general phenomenon known as Hyrum's Law: with enough consumers, every observable behavior of a system — even undocumented, even unintended — ends up depended on by someone). SemVer's version number is a claim about intent, not a verified guarantee.
- **It doesn't resolve diamond dependencies on its own** — SemVer tells you a version bump is safe, but if A wants C ≥ 2.0 and B wants C < 2.0, no version number alone resolves that; you need an actual compatible version to exist, or one of A/B to change.
- **It optimizes for infrequent, deliberate integration** (pin a version, upgrade occasionally, hope the version number's promise holds) — which concentrates the risk of an incompatibility into rare, large jumps rather than catching it early and in small increments.

## The alternative: live-at-head

**The idea**: instead of every consumer pinning a specific version of every dependency and upgrading occasionally, all consumers build against the current version of a dependency's source directly (via a shared, single-version repository — see `devops-practices`'s monorepo discussion) — there effectively is no version number to pin, because there's only ever one version in use across the whole graph at any moment.

**What this trades away**: no ability for a consumer to defer an upgrade — if a dependency changes, every consumer is building against the new version immediately, whether or not they're ready. This makes a dependency owner's *responsibility to avoid breaking consumers* much more direct and immediate (breakage is discovered right away, not months later at the next version-pin bump) — but it requires the infrastructure to actually verify nothing broke before or as part of that change (comprehensive test coverage across the whole dependency graph, and typically large-scale-automated-change tooling to fix consumers when a genuinely breaking change is unavoidable).

**When each model fits**: live-at-head fits an organization with the infrastructure to test the whole graph continuously and the tooling to push fixes broadly (the monorepo-at-scale model) — the diamond dependency problem structurally can't occur because there's only one version. SemVer with pinned versions fits the more common case: independent packages published to a public registry, consumed by unrelated organizations with no shared CI, no shared tooling, and no way to verify compatibility except the version-number promise. Most teams are in the second situation and should treat SemVer as a useful-but-imperfect signal, not a guarantee — test dependency upgrades before merging them, don't just trust the version bump.

## Practical takeaways

- **When debugging "SemVer said this was safe and it broke anyway,"** the actual cause is very likely Hyrum's Law (an undocumented behavior someone depended on) rather than the library author mis-tagging the version — worth checking what specifically changed against what the consumer actually relies on, not just re-reading the changelog.
- **When a diamond dependency conflict has no clean resolution,** that's a structural signal, not a tooling bug — one of the two conflicting dependencies needs to change, or the two libraries' version requirements need active reconciliation; no build tool can conjure a version that satisfies contradictory constraints.
- **When choosing module granularity**, default toward more, smaller, visibility-restricted targets over fewer, larger, open ones — the migration cost from coarse to fine grows with codebase size, so the earlier this default is applied, the cheaper it stays.
