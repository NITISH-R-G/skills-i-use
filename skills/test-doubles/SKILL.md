---
name: test-doubles
description: Reference for the specific techniques of substituting a real dependency in a test — fakes, stubs, mocks, and interaction testing — covering what each is precisely, when each is appropriate, and why state testing is generally preferable to interaction testing. Use this whenever the user is writing a test that needs to isolate code from a slow/external/nondeterministic dependency, deciding whether to mock something, debugging a brittle test that breaks on unrelated refactors, or asking "should I use a mock or a fake here."
---

# Test Doubles

A test double is any object substituted for a real dependency in a test. The umbrella term covers several distinct techniques that get used interchangeably in casual conversation ("just mock it") but behave very differently — picking the wrong one is a leading cause of tests that are brittle (break when the implementation changes, even though behavior didn't) or hollow (pass without actually verifying anything meaningful).

## The seam — where a double gets substituted

A double can only be substituted at a **seam** — a point where the code under test depends on an interface rather than a concrete implementation, so a different implementation can be swapped in without editing the code under test itself (see `codebase-design` for the general vocabulary). Code with no seams — a function that directly calls `new RealDatabase()` inline rather than receiving a `Database` interface — can't be tested with a double at all without first refactoring to introduce one. This is usually the actual first step when someone says "I can't test this without hitting the real database" — the problem is a missing seam, not a missing testing technique.

## The four techniques

### Stub

**What it is**: an object that returns canned answers to calls made during the test, with no real logic behind it.

```
stub.getUser(any) → returns a hardcoded User object, always
```

**When it's appropriate**: when the test only needs the dependency to return *something* plausible so the code under test can proceed — the dependency's own behavior isn't what's being tested, its return value is just an input to what actually is.

**The danger**: overusing stubs to force a code path (stubbing a method to throw, just to test the catch block) can produce a test that verifies the test's own setup more than it verifies real behavior — if the stub's canned response doesn't correspond to anything the real dependency would actually do, the test is checking a scenario that may not be reachable in practice.

### Mock (specifically: interaction testing)

**What it is**: an object that records what calls were made to it, so the test can assert *that specific calls happened*, often with specific arguments and in a specific order — the test's assertion is about the *interaction*, not about a return value or resulting state.

```
verify(mock).sendEmail(user.email, "welcome")  // asserting a call happened
```

**When it's appropriate**: specifically when the *fact that a call happened* is the actual behavior under test and there's no observable state change to assert on instead — sending an email, publishing an event, calling a payment gateway. If there's a side effect with no return value and no state you can inspect afterward, interaction testing (asserting the call happened) may be the only way to verify the behavior at all.

**Why it's the double to reach for last, not first**: interaction tests couple the test to the *implementation detail* of exactly how the code under test calls its dependency — which method, in what order, with which exact arguments. Refactor the implementation to achieve the identical observable behavior a different way (call the methods in a different order, batch two calls into one) and an interaction test breaks even though nothing a user or caller would notice actually changed. This is the single biggest source of brittle tests that force reviewers to update tests on every refactor, training everyone to treat test failures as noise instead of signal.

### Fake

**What it is**: a working, simplified implementation of the real dependency's interface — not canned answers, actual logic, just lighter-weight (an in-memory database implementing the same interface as the real one, storing data in a hashmap instead of over a network on disk).

**When it's appropriate**: when a lightweight-but-real implementation exists or is worth building — a fake lets the test exercise real logic (the in-memory database actually stores and retrieves data, actually enforces uniqueness constraints if the real one does) without the cost/flakiness of the real thing.

**Why fakes are usually the best double, when available**: unlike a stub (canned, no real logic) or a mock (asserts implementation details), a fake behaves close enough to the real dependency that a test using it is verifying something close to real behavior — state testing against a fake, not interaction testing against a mock. The trade-off is cost: someone has to write and maintain the fake, and it needs its own tests to verify it actually behaves like the real thing (an incorrect fake is worse than no fake — every test built on it inherits the wrong assumption).

**Who should write the fake**: ideally the team that owns the real dependency, since they know its actual contract and can keep the fake in sync as the real implementation evolves. A fake written by a consumer, guessing at behavior, tends to drift from reality over time.

### Real implementation

**Often the right default when it's fast and deterministic enough to use directly** — not every dependency needs a double at all. A pure in-memory data structure, a deterministic calculation, a small well-behaved library — using the real thing is simpler than introducing any double, and it's the only option with zero risk of the double's behavior diverging from reality.

**When it's not appropriate**: the real implementation is slow (network calls, disk I/O), nondeterministic (current time, random values, race conditions), has expensive side effects (sends a real email, charges a real credit card), or is hard to get into a specific state for the test (needs a specific error condition that's hard to trigger for real).

## Decision order

Prefer, in this order, for any given dependency:

1. **Use the real implementation** if it's fast, deterministic, and side-effect-free enough for a small/unit-scope test.
2. **Use a fake** if the real implementation doesn't qualify but a lightweight, well-maintained fake exists (or is worth building because many tests will use it).
3. **Use a stub** if you just need a specific return value and there's no meaningful behavior to fake.
4. **Use interaction testing (a mock)** only when the behavior under test genuinely has no observable state to assert on — the call itself is the only thing to verify.

This order isn't arbitrary — it's ordered by how closely the double's behavior tracks the real dependency's behavior, which is what determines how much a passing test actually tells you.

## State testing vs. interaction testing — the core preference

**State testing**: call the code under test, then assert on the resulting state (a return value, a change in a real or faked collaborator's observable state). Tests *what happened*.

**Interaction testing**: assert on *how* the code under test called its collaborators. Tests *how it happened*.

**Prefer state testing whenever a meaningful state change exists to assert on.** State tests survive refactoring that preserves behavior — reorder internal calls, change which helper method does the work, and a state test still passes because the observable outcome is unchanged. Interaction tests don't survive that kind of refactor, because they were pinned to the implementation detail of the call sequence itself, not the outcome.

**Practical test for whether interaction testing is actually warranted**: ask "if I changed how this achieves the same visible result, should this test still pass?" If yes, and the current test would break anyway, that's a sign the test is over-specifying the implementation — look for a state assertion instead. If the answer is genuinely no (the fact that this specific external call happens is itself the contract, e.g., "we must call the audit log exactly once per transaction"), interaction testing is the correct tool for that specific case.

## Best practices for interaction testing, when it is appropriate

- **Assert on calls that matter to the contract**, not incidental implementation calls — mocking a private helper method's internals over-specifies; mocking the boundary to an external system (payment gateway, email service) is testing the actual contract.
- **Don't over-verify** — asserting the exact argument on every call, including incidental ones, makes the test brittle to changes that don't affect behavior. Assert what the test is actually about.
- **Keep the double close to the real contract** — if the real dependency's interface changes and the mock/stub isn't updated to match, the test can pass while the real integration is broken. This is the general risk of any double: it's only as trustworthy as its fidelity to the real thing.
