---
name: twelve-factor-apps
description: Reference for the twelve-factor app methodology for building cloud-native, horizontally-scalable services — config, dependencies, statelessness, disposability — plus where the original 12 factors need updating for containerized/Kubernetes-era deployment. Use this whenever the user is designing a new service meant to run in the cloud or in containers, debugging environment-specific bugs ("works on my machine"), setting up config/secrets management, or reviewing a service for cloud-readiness.
---

# Twelve-Factor Apps

The twelve-factor methodology is a set of concrete, checkable practices for building services that behave predictably across environments (dev, staging, prod) and scale horizontally without surprises. Each factor exists to eliminate one specific, recurring class of "works here, breaks there" bug.

## The factors that matter most in practice

**Config lives in the environment, not in code.** Anything that varies between deploys (database URLs, API keys, feature flags) belongs in environment variables, never hardcoded or checked into version control alongside the code. The test: could this codebase be made open source right now without exposing any secret or environment-specific value? If not, something that should be config is currently baked into code.

**Why this specifically, not just "good practice"**: config-in-code means the same build artifact behaves differently depending on which copy of the source you're looking at, which defeats the goal of one build being promotable unchanged from staging to production (see "build, release, run" below) — and it's how secrets end up in git history, which is nearly impossible to fully scrub once committed.

**Dependencies are explicitly declared and isolated**, never assumed to exist on the host system. A service that "works" because it happens to rely on a system library someone once installed manually on the production box will fail the moment it's deployed to a fresh environment that wasn't set up the same undocumented way. Explicit dependency manifests (see `build-systems-and-dependencies`) plus isolation (containers, virtual environments) make "works on my machine" a solvable problem rather than a recurring mystery.

**Processes are stateless; anything persistent goes in a backing service.** A process can be killed and restarted at any moment (see disposability below) — if it's holding session state, uploaded files, or in-memory data that's supposed to survive, that data is gone when the process cycles, and worse, is inconsistent across instances if you're running more than one. State belongs in a database, cache, or object store — a backing service reachable by any instance of the app — never in the app process's own memory or local disk.

**Backing services are attached resources, swappable via config.** A database, cache, or message queue should be reachable purely through a config-supplied URL/credential, with no code distinguishing "the local dev database" from "the production database" beyond that config value. This is what makes promoting the same build from staging to production safe — the code doesn't know or care which environment's resources it's talking to; only the config changes.

**Build, release, run are strictly separate stages.** Build compiles/bundles code into an artifact; release combines that artifact with environment-specific config; run executes a release. Critically, a build artifact should be usable to create *any* environment's release — you build once, then release it against staging config, then (the *same* build) release it again against production config. A pipeline that rebuilds the code separately for each environment reintroduces exactly the "different code path per environment" risk the whole methodology is trying to eliminate — if staging and production were built from separately-triggered builds, you can no longer be certain they're running identical code.

**Processes are disposable — fast startup, graceful shutdown.** A process should be startable and killable at a moment's notice, without ceremony, and should handle a shutdown signal by finishing in-flight work and exiting cleanly rather than dying mid-operation. This is what makes horizontal scaling (add or remove instances on demand) and resilient deployment (kill an unhealthy instance and replace it) actually safe operations rather than risky ones — an app that takes 90 seconds to start or leaves work half-done on shutdown makes both scaling and deployment fragile.

**Dev/prod parity — keep environments as similar as possible.** The gap between what a developer runs locally and what runs in production (different database engine, different OS, code deployed hours or days after being written) is where "works in dev, breaks in prod" bugs live. Minimizing that gap — same backing service types in dev as prod (even if smaller), fast and frequent deploys rather than infrequent big-bang releases, the same people writing and deploying code rather than a separate ops team translating — shrinks the surface area for environment-specific surprises.

**Logs are treated as event streams, not files the app manages.** The app writes logs to stdout/stderr, unbuffered — and lets the execution environment (the container runtime, the orchestrator) handle routing, storage, and rotation. An app that manages its own log files (rotation, archival, where they're stored) is taking on infrastructure concerns that belong to the platform, and that self-managed logic breaks in exactly the kind of environment-specific way the whole methodology tries to avoid.

## Where the original 12 need updating for the container/Kubernetes era

The methodology predates widespread container orchestration, and a few factors benefit from restating in that context rather than following the letter of the original:

**Add: telemetry as a first-class concern, not an afterthought.** The original 12 don't dedicate a factor to metrics/tracing/health checks — in a Kubernetes-era deployment, exposing structured health-check endpoints (liveness/readiness probes) and metrics (for the observability practices in `observability-and-monitoring`) is as fundamental as anything on the original list, because the orchestrator actively uses this information to make scheduling and restart decisions, not just for human dashboards.

**Add: explicit API contracts / service definitions.** Modern deployments frequently involve many small services calling each other (see `microservices-patterns-and-antipatterns`) — a factor the original list didn't need to emphasize as heavily, since it assumed a more monolithic-ish single-app context. Treat the service's API contract as itself a versioned, explicitly declared artifact, not an implicit convention.

**Reconsider: admin/management processes.** The original factor suggests running one-off admin tasks (migrations, console scripts) as one-off processes using the same codebase/config as the long-running app. In a Kubernetes-era world this maps onto Jobs/init containers rather than an ad hoc shell into a running pod — the *principle* (same codebase, same config-driven environment, not a bespoke separate script) still holds; the *mechanism* has a more specific modern implementation.

**Reconsider: port binding.** The original factor emphasizes the app being self-contained and binding its own port rather than relying on runtime injection of a web server. In container orchestration, the platform still expects the app to bind a port and be otherwise self-contained — this factor holds up essentially unchanged, just executed through a container's exposed port rather than a bare process.

## Practical checklist

- Can this service be deployed to a fresh environment with zero manual setup steps beyond supplying config?
- Is there anything checked into version control that varies between dev/staging/prod, or is a secret?
- Does the app hold any state in process memory or local disk that would be lost if the process restarted right now?
- Was the artifact running in production built by the same build step that created the artifact tested in staging — or was it rebuilt separately?
- Does the app expose health-check endpoints an orchestrator can actually use to make restart/scheduling decisions?
- How long does the app take to start, and does it handle a shutdown signal gracefully or die mid-request?
