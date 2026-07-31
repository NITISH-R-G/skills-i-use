---
name: container-infrastructure
description: Reference for immutable infrastructure, container security fundamentals, and core Kubernetes concepts (pods, deployments, services, health checks). Use this whenever the user is containerizing an application, setting up or debugging a Kubernetes deployment, deciding between mutable server management and immutable infrastructure, or reviewing container configuration for security issues.
---

# Container Infrastructure

## Immutable infrastructure

**The idea**: once a server/container instance is deployed, it's never modified in place — any change (a code update, a config change, a patch) is deployed by building a *new* image/instance and replacing the old one, never by SSHing in and changing something on a running instance.

**Why this beats the mutable alternative**: a server that's been manually patched, tweaked, and hotfixed over months accumulates **configuration drift** — its actual state diverges from what any documentation or provisioning script says it should be, until no one can confidently reproduce it from scratch. When that server eventually needs replacing (hardware failure, scaling up), reconstructing its exact accumulated state is often impossible — this is "snowflake server" territory, and it's a direct cause of "it works in prod but nowhere else, and we don't know why."

**What immutability buys you concretely**:
- **Reproducibility** — a fresh instance built from the current image is guaranteed identical to every other instance built from that image, because there's no path for it to have drifted.
- **Reliable rollback** — reverting is redeploying the previous image, not trying to manually undo whatever changes were made in place (which requires remembering what those changes were).
- **Confidence in "it works"** — because staging and production were built from the same image (see `twelve-factor-apps`'s build/release/run separation), a service verified in staging carries much stronger evidence about production than in a mutable world where the two might have quietly diverged.

**The trade-off**: rebuilding and redeploying a whole image for a one-line config change feels heavier than SSHing in and editing a file — but that felt convenience is exactly the mechanism that produces drift. Fast, automated build/deploy pipelines are what make immutability practical rather than a heavyweight burden — the discipline depends on the pipeline being fast enough that "just rebuild it" is genuinely the path of least resistance, not a chore people route around.

## Container security fundamentals

**Run as a non-root user inside the container.** A container's process, by default, often runs as root — if an attacker compromises the application, root-inside-the-container is a meaningfully worse starting position for them than a restricted user, particularly combined with any container-escape vulnerability. Explicitly set a non-root user in the image unless there's a specific, understood reason not to.

**Minimize the base image.** A full OS-based image carries a large surface of packages, libraries, and shells the application doesn't need — each one is a potential vulnerability the image inherits whether the app uses it or not. Minimal or distroless base images reduce this attack surface directly, at the cost of losing convenient debugging tools inside the container (a trade-off worth making deliberately, with a documented debugging strategy that doesn't depend on shelling into production).

**Never bake secrets into the image.** A secret (API key, credential, private key) committed into an image layer is recoverable by anyone who can pull that image, even if a later layer appears to delete it — image layers are typically append-only history, not a clean overwrite. Secrets belong in runtime-injected config (environment variables from a secrets manager, mounted secret volumes), never in the image itself. This is a container-specific instance of the config-in-environment principle from `twelve-factor-apps`.

**Scan images for known vulnerabilities, and do it in the pipeline, not after deploy.** Base images and dependencies accumulate known CVEs over time even if the application code never changes — a scan integrated into CI (see `static-analysis`'s placement-in-the-workflow principle) catches this before a vulnerable image ships, rather than relying on a separate, easily-neglected periodic audit.

**Set resource limits explicitly.** A container with no CPU/memory limit can consume unbounded resources on its host — whether from a legitimate bug (a memory leak) or a malicious workload, an unlimited container can starve every other workload on the same node. Explicit limits contain the blast radius of either case to the one container.

**Read-only filesystem where possible.** A container that doesn't need to write to its own filesystem at runtime should be run with a read-only root filesystem — this removes an entire class of exploit (an attacker writing a malicious file to disk and executing it) as a possibility, not just something to detect after the fact.

## Kubernetes — core concepts

**Pod**: the smallest deployable unit — one or more tightly-coupled containers sharing network and storage. Usually one container per pod in practice; multiple containers in a pod are for genuinely coupled helper processes (a sidecar), not a general-purpose way to group unrelated services.

**Deployment**: declares the desired state for a set of identical pod replicas (which image, how many copies, resource limits) and manages rolling out changes to that state — you declare "I want 5 replicas of image X," and the Deployment controller continuously works to make reality match that declaration, replacing failed pods automatically. This is the concrete embodiment of immutable infrastructure's "declare desired state, don't hand-edit running instances" principle.

**Service**: a stable network identity/address in front of a changing set of pods. Pods are ephemeral — they get killed and replaced routinely — so nothing should address a pod directly by its own (constantly changing) address; a Service provides a fixed endpoint that routes to whichever pods currently match its label selector, regardless of individual pod churn underneath.

**Health checks — liveness and readiness probes, and the distinction between them**:
- **Liveness probe**: "is this container in a state where it should be killed and restarted?" A failing liveness probe causes Kubernetes to restart the container. Use it for detecting a genuinely stuck/deadlocked process — not for transient conditions that will resolve on their own, since restarting won't fix a transient problem and just adds churn.
- **Readiness probe**: "should this container currently receive traffic?" A failing readiness probe removes the pod from a Service's routing targets *without* killing it — used for a pod that's temporarily unable to serve (still warming up a cache, temporarily overloaded, waiting on a dependency) but will recover without needing a restart.

**The common mistake**: conflating the two, or implementing only one. A liveness probe that's too aggressive (flags transient slowness as "dead") causes unnecessary restart churn, actively making a temporary slowdown worse by adding restart overhead on top. A missing readiness probe means a pod receives traffic the moment it starts, even before it's actually ready to serve correctly — causing a wave of errors during every rollout as new pods get traffic before they've finished initializing.

**Rolling updates**: replace old pods with new ones incrementally (not all at once), keeping some minimum number of ready pods serving traffic throughout — this is what makes a deploy low-risk: if the new version's health checks start failing partway through the rollout, the deployment can be halted or rolled back with only a fraction of traffic ever having reached the bad version, not all of it at once.

## Practical checklist

- Does the deployment pipeline make "rebuild and redeploy" fast and automated enough that no one is tempted to hand-edit a running instance?
- Does the container run as non-root, with a minimal base image, and no secrets baked into any layer?
- Are images scanned for known vulnerabilities as part of CI, not as a separate afterthought?
- Does every deployed workload have explicit resource limits set?
- Are liveness and readiness probes both configured, and do they actually measure the right thing each is meant to measure (stuck vs. temporarily-not-ready)?
