---
name: testing-strategy
description: Reference for structuring a test suite well — test size (small/medium/large) versus test scope (unit/integration/e2e, a different axis), the testing pyramid, and the real limits of code coverage as a metric. Use this whenever the user is deciding what kind of test to write for something, designing a test suite's overall shape, arguing about coverage targets, debugging a slow or flaky test suite, or asking "what should I even test here."
---

# Testing Strategy

Two questions get conflated constantly: **how big/slow/isolated is this test** (size) and **how much of the system does it exercise** (scope). They're different axes, and a healthy test suite needs a deliberate shape along both — not just "write tests," but a specific, defensible distribution of sizes and scopes.

## Test size — a resource-usage axis

Size is about the *constraints* a test runs under, not what it verifies:

| Size | Runs on one machine? | Can use network/DB/disk? | Can sleep/block? | Speed |
|---|---|---|---|---|
| **Small** | Yes, single process | No — everything is faked or in-process | No | Milliseconds |
| **Medium** | Yes, single machine, multiple processes allowed | Local only (localhost DB, local files) | Limited | Seconds |
| **Large** | No — can span machines | Yes, real network calls allowed | Yes | Seconds to minutes+ |

A test's size determines how *often* it can run in the loop. Small tests are cheap enough to run on every save, every commit, hundreds of times a day — that's what makes `tdd`'s red-green loop viable. Large tests are expensive enough that they typically run on a schedule or in CI only, not in the tight inner loop.

**The common mistake**: writing a test that's logically "unit-like" (tests one function) but accidentally large (hits a real database because the code under test wasn't given a seam to fake it through). It looks small in intent and behaves large in practice — slow, flaky, blocking the fast loop it was meant to serve. This is almost always a design problem in the code under test (see `test-doubles` and `codebase-design` on seams), not a testing problem.

## Test scope — a what-does-it-exercise axis

Scope is orthogonal to size — it's about how much of the system a test's assertions actually span:

- **Unit** — one function/class/module in isolation, collaborators faked or stubbed.
- **Integration** — two or more real components interacting (a real database, a real second service), verifying they cooperate correctly at the boundary.
- **End-to-end (E2E)** — the whole system, or a realistic slice of it, exercised the way a real user or client would.

A test can be small-scope-unit (the common case), or medium-scope-integration, or large-scope-e2e — size and scope correlate loosely (E2E tests are almost always large; unit tests are almost always small) but they're answering different questions, and conflating them is how "we have 90% unit test coverage" ends up masking a system with zero tests of whether the pieces actually work together.

## The testing pyramid — the intended shape

```
        /\
       /E2E\      <- few: slow, brittle to any change, but catch real integration gaps
      /------\
     /  Integ  \  <- some: verify real component boundaries
    /------------\
   /     Unit      \ <- many: fast, precise, pinpoint failures to one unit
  /------------------\
```

**Why this shape, not an inverted one**: a failing unit test tells you exactly which function broke; a failing E2E test tells you *something* broke somewhere in a large surface, and debugging it means narrowing down manually. E2E tests are also the most expensive to maintain — they break on unrelated changes (a UI element moves, a timing assumption shifts) at a much higher rate than a well-isolated unit test does. The pyramid shape optimizes for fast, precise, cheap-to-maintain signal at the base, with a thin layer of expensive-but-necessary broad-coverage tests at the top catching what unit tests structurally can't (real integration failures, real infra misconfigurations).

**An inverted pyramid (the "ice cream cone" anti-pattern)** — mostly E2E tests, few unit tests — is a common organic failure mode: E2E tests feel like they're testing "the real thing" so they get written preferentially, while unit tests require design work (seams, faking) that gets skipped under time pressure. The result is a suite that's slow to run, flaky, and unhelpful at pinpointing failures — exactly the properties that erode a team's trust in its own tests over time.

## The Beyoncé Rule

*"If you liked it, you should have put a test on it."* Any behavior you care about not silently breaking needs its own test asserting it — you don't get to rely on some other, unrelated test happening to also catch it. A behavior with no test that specifically pins it down is a behavior that's already lost the moment someone refactors the code around it — there's no signal telling them they broke your thing, because nothing was actually testing for it.

**Practical application**: when reviewing a PR that changes behavior, ask specifically "is there a test that would fail if this behavior regressed?" — not "does the test suite still pass" (that only tells you existing behaviors weren't broken, not that the new one is locked in).

## Code coverage — what the number actually tells you, and doesn't

**What it measures**: the percentage of lines/branches executed by the test suite. That's it — nothing about whether the test *asserted* anything meaningful about what it executed.

**Why 100% coverage doesn't mean "well tested"**: a test that calls a function and asserts nothing about its return value gives that function coverage credit with zero verification of correctness. Coverage is a *floor* signal (code with 0% coverage is definitely under-tested) not a *ceiling* signal (code with 100% coverage is not necessarily well-tested).

**What coverage is actually good for**: finding code that has *no* tests exercising it at all — a genuinely useful signal for spotting gaps — and tracking trend direction (coverage dropping over time on a codebase that used to be well-tested is worth investigating). It's a much weaker signal used as a target number itself: chasing a coverage percentage as a goal tends to produce tests written to hit lines, not tests written to verify behavior, which is optimizing for the metric instead of the thing the metric was supposed to proxy for (Goodhart's Law in miniature).

**Practical stance**: use coverage to find untested code, not as a quality gate on its own. Pair it with the Beyoncé Rule test — "does a test exist that would fail if this specific behavior broke" — which coverage alone can't answer.

## Deciding what to write, practically

| Situation | Reach for |
|---|---|
| Verifying one function/class's logic in isolation | Small, unit-scope |
| Verifying two real components cooperate correctly at a boundary you own | Medium, integration-scope |
| Verifying the system behaves correctly the way a real client sees it, for a critical path | Large, E2E-scope — sparingly, for the paths that matter most |
| A behavior you specifically care about not regressing | Whatever size/scope reaches it directly — apply the Beyoncé Rule, don't assume another test covers it |
| Debugging a slow, flaky suite | Check the pyramid shape — an inverted pyramid (too many large/E2E tests) is the most common root cause |

This complements `tdd` (the red-green authoring loop, mostly operating at small/unit scope) and `test-doubles` (the specific techniques — fakes, stubs, mocks — for keeping a test's size small while still exercising real logic).
