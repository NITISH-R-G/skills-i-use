---
name: devsecops
description: Reference for integrating security and compliance into the delivery pipeline rather than treating them as a late, separate gate — shifting security left, automating compliance evidence, and threat modeling as an ongoing practice. Use this whenever the user is setting up security scanning in CI/CD, dealing with a compliance requirement (SOC2, PCI, HIPAA-style controls), designing a threat model for a new feature, or debugging why security review is a bottleneck late in the release process.
---

# DevSecOps

The traditional failure mode this addresses: security and compliance review happening as a separate, late gate — after the code is written, sometimes after it's already deployed to staging — where a finding means expensive rework on already-"finished" work, and the review team becomes a bottleneck everyone routes around when possible. DevSecOps is the same "shift left" idea already covered for testing (`tdd`, `testing-strategy`) and static analysis (`static-analysis`), applied to security specifically: catch problems when they're cheap to fix, not after they're expensive to fix.

## Shifting security left — where in the pipeline it actually belongs

Mirroring the placement logic from `static-analysis`, security checks belong as early as their speed and confidence allow:

| Stage | What belongs here |
|---|---|
| **IDE / pre-commit** | Secret-scanning (catch a credential before it's even committed, since once it's in git history it's expensive to fully remove), basic linting for known-dangerous patterns (unsanitized SQL construction, etc.) |
| **CI, every push** | Dependency vulnerability scanning (known CVEs in third-party packages — see `build-systems-and-dependencies`), static application security testing (SAST) for code-level vulnerability patterns, container image scanning (see `container-infrastructure`) |
| **Pre-deploy / staging** | Dynamic application security testing (DAST) — probing the running application for vulnerabilities that only manifest at runtime, not visible from source alone |
| **Ongoing, scheduled** | Periodic full-codebase or full-infrastructure audits, penetration testing — the expensive, thorough checks that don't fit in a per-commit loop |

**Why the ordering matters, not just the presence of each check**: a secret leaked into a commit and caught at CI time (minutes later) is a relatively contained cleanup; the same secret caught by a scheduled weekly audit has had days to be pulled by a dependent process, cached somewhere, or (worst case) exploited — the same finding, wildly different cost, purely as a function of when it was caught. This is the general `static-analysis` placement principle, with genuinely higher stakes here than for an ordinary lint finding.

**The trap to avoid**: treating "shift left" as "add a slow, thorough scan to the fast pre-commit loop." A full dependency audit or a deep SAST pass that takes 20 minutes doesn't belong in a pre-commit hook — putting it there either gets bypassed constantly (killing its value) or trains developers to dread committing. Match each check's actual cost/confidence to the right stage, the same discipline `static-analysis` covers for non-security checks — security checks aren't exempt from that trade-off just because the stakes are higher.

## Security as everyone's job, not a separate team's gate

**Why a security team, by itself, can't scale with the org**: a small central security team reviewing every change becomes a bottleneck the moment the org's shipping velocity outpaces the team's review capacity — and unlike code review generally (which can be distributed across many engineers), a security team's specialized knowledge doesn't distribute the same way by default. The DevSecOps answer is automating what can be automated (the CI-stage checks above) so the security team's limited human attention concentrates on what genuinely needs expert judgment (architecture-level threat modeling, novel attack surfaces) rather than getting consumed by things a scanner could have caught.

**This mirrors `engineering-culture`'s point about testing and code quality being a whole-team responsibility, not a separate QA team's job** — the same shift, applied to security: automated checks plus a security-aware engineering culture scale further than a gatekeeping team ever can alone, and the gatekeeping-team model tends to produce exactly the late, expensive, resented review process this whole practice exists to avoid.

## Threat modeling as an ongoing practice, not a one-time document

**The common failure**: a threat model written once, early in a project, that's never revisited — accurate for the system as it existed at the time, silently stale the moment the architecture meaningfully changes (a new external integration, a new data flow, a new trust boundary). A stale threat model gives false confidence, the same way stale documentation does (see `documentation-practices`) — worse, arguably, because the false confidence is specifically about what's been checked for security risk.

**A lightweight, repeatable framing** (STRIDE, one common structure) for revisiting threat modeling whenever a design changes meaningfully, not just once at project inception: for each component and each trust boundary it crosses, ask whether the design is vulnerable to **S**poofing (is identity verified), **T**ampering (can data be modified in transit or at rest without detection), **R**epudiation (can an action be traced to who performed it), **I**nformation disclosure (can unauthorized parties read data they shouldn't), **D**enial of service (can availability be degraded), **E**levation of privilege (can a lower-privilege actor gain higher privilege).

**Practical trigger for re-running threat modeling**: any design review that introduces a new trust boundary (a new external API integration, a new class of user with different permissions, a new place user input crosses into a privileged operation) is a natural, low-overhead point to ask the STRIDE questions again for the specific new surface — this keeps threat modeling proportional and ongoing rather than a heavyweight annual exercise that's disconnected from when the architecture actually changes.

## Compliance as continuous evidence, not an annual scramble

**The failure mode this replaces**: compliance evidence (SOC2, PCI-DSS, HIPAA-style controls, or similar) gathered manually, under time pressure, once a year before an audit — screenshots, manually-exported logs, someone's memory of what changed — which is both extremely labor-intensive and, because it's assembled after the fact from imperfect records, not actually a reliable account of whether controls were followed continuously throughout the year.

**The DevSecOps alternative**: bake evidence-generation into the pipeline itself, so compliance evidence is a continuous byproduct of normal operations rather than a separate annual project. Concretely: access control changes logged automatically as part of the infrastructure-as-code pipeline (not tracked manually), every production deploy automatically recording what changed, who approved it, and what tests/scans passed (rather than reconstructing this from memory later), and security scan results retained as a continuous audit trail (rather than only the pass/fail gate result being visible, with the underlying evidence discarded).

**Why this is a better trade even ignoring audit season**: continuous evidence-generation catches control failures in near-real-time (a required approval step that got skipped shows up immediately in the pipeline record) rather than only being discovered — or worse, not discovered at all — during the once-a-year audit review. The audit becomes a matter of pulling existing records rather than an annual fire-drill of trying to reconstruct a year's worth of history from scattered, incomplete sources.

## Practical checklist

- Are security checks placed in the pipeline at a stage matched to their actual speed/confidence, or is a slow scan sitting in a fast pre-commit path (or a fast check missing entirely from CI)?
- Is a leaked secret caught within minutes (pre-commit/CI), or only during a periodic audit days or weeks later?
- Does compliance evidence get generated continuously as a pipeline byproduct, or assembled manually once a year under audit pressure?
- Has the threat model been revisited since the last meaningful architecture change (a new integration, new trust boundary, new user class) — or is it accurate only for how the system looked at project inception?
- Is security review structured to concentrate a small expert team's attention on genuinely novel risk, with routine/known-pattern checks automated away from them?
