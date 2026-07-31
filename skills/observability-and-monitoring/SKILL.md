---
name: observability-and-monitoring
description: Reference for monitoring and observability in distributed systems — the four golden signals, designing alerts that get acted on instead of ignored, and anomaly detection basics. Use this whenever the user is setting up monitoring/alerting for a service, debugging why alerts are noisy or missed real incidents, deciding what metrics actually matter for a system, or designing dashboards.
---

# Observability and Monitoring

Monitoring exists to answer two different questions, and conflating them produces bad systems: "is something wrong right now" (alerting — needs to be fast, precise, and rare) and "what happened, historically, when debugging" (observability — needs to be detailed and explorable, and can be slower to query). Build for both, but don't let the second question's need for detail pollute the first question's need for precision.

## The four golden signals

For any user-facing service, these four signals catch the overwhelming majority of real problems, and are the right *starting* set before adding anything more specific:

1. **Latency** — how long requests take. Track the distribution (p50/p95/p99), not just the average — a mean can look fine while a meaningful fraction of users have a terrible experience; the tail is where most real user pain lives. Separate latency of successful requests from failed ones — a fast error is a different problem than a slow success, and averaging them together hides both.
2. **Traffic** — how much demand the system is serving (requests/sec, or a domain-appropriate unit). Needed as context for the other three signals — a latency spike might just be a traffic spike, not a regression.
3. **Errors** — the rate of requests failing, explicitly (5xx) or implicitly (a 200 response with the wrong content, a business-logic failure reported as success). The implicit case is easy to miss — a service can look perfectly healthy on HTTP status alone while failing at the business-logic layer.
4. **Saturation** — how "full" the system is relative to capacity (CPU, memory, queue depth, connection pool usage). The signal that predicts problems *before* they show up in the other three — a system at 95% memory saturation is a warning sign even while latency and error rate still look fine, because the failure when it comes will likely be sudden.

**Why these four specifically**: together they answer "is this working" (traffic, errors), "is this working well" (latency), and "is this about to stop working" (saturation) — a minimal set that catches most real incidents without instrumenting every possible internal detail up front. Add system-specific signals on top once these four are solid, not instead of them.

## Designing alerts that get acted on

**The core failure mode — alert fatigue**: the same dynamic covered in `static-analysis` for lint noise applies here, with higher stakes — an alert that fires on conditions that don't actually need human action trains on-call to treat pages as noise, and the training generalizes to genuine emergencies mixed in with the noise. An on-call rotation that's learned to acknowledge-and-ignore has already lost most of the value alerting was supposed to provide.

**Alert on symptoms, not causes.** "Latency p99 > 2s" (a symptom users actually experience) is a better alert condition than "CPU > 80%" (a cause that may or may not actually be hurting anyone right now). Cause-based alerts fire on conditions that are sometimes fine (a batch job legitimately spiking CPU) and sometimes not, forcing the on-call engineer to investigate every time to find out which — symptom-based alerts fire specifically when something is actually degraded for users, which is the thing worth waking someone up for.

**Every alert should be actionable.** If an alert fires and the correct response is "acknowledge and do nothing, this always happens" — that's not an alert, that's noise wearing an alert's clothing, and it should be deleted or its threshold fixed, not tolerated. A useful test: for each alert, can you name the specific action the responder should take when it fires? If not, the alert isn't ready to page anyone.

**Route by urgency, not by whoever owns the metric.** A slow-burning issue that can wait until business hours (rising error rate that's still low in absolute terms) shouldn't page someone at 3am the same way an outage does — separate paging (wakes a human immediately) from ticket-generating alerts (reviewed during business hours) based on genuine urgency, not just on whether monitoring noticed something.

**Set thresholds from the system's actual behavior, not round numbers.** A threshold picked because "80% feels like a reasonable saturation limit" without checking what this specific system's actual failure point is will either fire too early (this system runs fine at 85%) or too late (this system starts failing at 70%). Baseline the system's real behavior under load before setting a threshold that's supposed to predict trouble.

## Anomaly detection — when static thresholds aren't enough

**The limitation static thresholds hit**: a fixed threshold ("alert if traffic drops below X") works for systems with stable, predictable traffic patterns, but fails for anything with legitimate variation — daily/weekly cyclicality (traffic naturally drops overnight), seasonal patterns (a retail system's Black Friday traffic would trip a threshold set from a normal Tuesday). A static threshold tuned to avoid false positives during normal variation ends up too loose to catch a real anomaly of similar magnitude.

**What anomaly detection buys you instead**: comparing current behavior to a learned baseline of *what's normal for this time/day/season*, rather than to one fixed number — so a genuine deviation from the expected pattern triggers an alert even if the absolute value would have looked fine against a naive fixed threshold, and expected cyclical variation doesn't trigger false alarms.

**Where it's worth the complexity, and where it isn't**: anomaly detection adds real complexity (a model to maintain, a training/baseline period, its own tuning problems) — worth it specifically for metrics with genuine cyclical/seasonal structure where static thresholds have a demonstrated false-positive or false-negative problem. For a metric with genuinely stable expected behavior (error rate should always be near zero, regardless of time of day), a simple static threshold is more transparent, easier to debug when it misfires, and usually just as effective — don't reach for anomaly detection as a default; reach for it when a specific metric's natural variability has already made static thresholds demonstrably unworkable.

## Dashboards — designed for a specific question, not everything at once

The same principle from `documentation-practices` about matching content to audience applies to dashboards: a dashboard trying to show every metric to every possible viewer ends up serving no one — the on-call engineer during an incident needs a small, fast, symptom-focused view (the four golden signals, front and center) to triage quickly; a capacity-planning review needs longer-horizon trend views that would just be noise during an active incident. Build separate dashboards for separate questions rather than one dashboard everyone's expected to parse differently depending on why they're looking at it.

## Practical checklist

- Do the four golden signals exist for every user-facing service, at minimum?
- For every alert that pages someone: is there a specific, named action the responder should take? If not, fix or delete it.
- Are alerts symptom-based (user-visible degradation) rather than cause-based (an internal metric crossing a number) wherever possible?
- Has anyone actually measured this rotation's false-positive rate, or is "seems fine" the only evidence?
- Do dashboards serve one specific question each, or is there one sprawling dashboard trying to answer everything?
