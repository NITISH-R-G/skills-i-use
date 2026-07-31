---
name: refactoring-catalog
description: Catalog of code smells and the matching refactoring techniques that fix them, plus the discipline of refactoring safely (small steps, test-covered, behavior-preserving). Use this whenever the user asks to clean up or refactor code, describes code that's hard to change or understand, wants to name what's structurally wrong with a piece of code, or you notice a recognizable smell (long method, duplicated logic, feature envy, primitive obsession, etc.) while reading code for any other task.
---

# Refactoring Catalog

Refactoring is changing a program's internal structure without changing its observable behavior — same inputs produce the same outputs, every time, before and after. That constraint is what separates refactoring from "rewriting while also maybe fixing bugs and adding features" — mixing those in one commit destroys the ability to verify that behavior didn't change, and is how "quick cleanup" branches accumulate hidden risk.

## The discipline, before the catalog

1. **Never refactor without test coverage over the code being touched.** If coverage is missing, writing characterization tests (tests that pin down current behavior, whatever it is) is the first step — not a nice-to-have before the "real" refactoring starts.
2. **Small steps, verified between each one.** Each individual refactoring (rename, extract, inline) should be small enough that if something breaks, it's obvious which step broke it. Running the test suite after every step, not after the whole session, is what makes this safe rather than theoretical.
3. **Refactor and feature work are separate commits.** "While I was in there I also refactored X" commits make review harder (is this diff a behavior change or not?) and make bisection harder later. Keep them separate even when it's tempting to combine.
4. **Refactor with a reason, not on a schedule.** The classic trigger is "preparatory refactoring" — the code needs to be a certain shape before the next feature can be added cleanly ("make the change easy, then make the easy change"). Refactoring with no concrete next step it's enabling tends to wander and lose its stopping point.

This pairs directly with `codebase-design` (deep modules, seams, interfaces) — that skill covers *what shape code should be in*; this one covers *how to recognize when it isn't, and how to move it there safely*.

## Code Smells → Matching Refactorings

### Bloaters (things that have grown too large to reason about)

**Long Method** — a method doing too much to hold in your head at once, usually recognizable because it needs a lot of comments to explain its sections.
→ **Extract Method**: pull a cohesive chunk into its own well-named method. The name replaces the comment that used to explain the chunk. Repeat until each method does one describable thing.

**Large Class** — a class with too many responsibilities, too many fields, too many methods.
→ **Extract Class**: identify a subset of fields/methods that form a cohesive sub-responsibility and move them to a new class, referenced by the original.

**Long Parameter List** — a method signature with so many parameters that call sites are hard to read and easy to get wrong (especially same-typed parameters in the wrong order).
→ **Introduce Parameter Object**: group related parameters that travel together into a single object. Also surfaces a missing domain concept — if `startDate, endDate` travel together everywhere, that's a `DateRange` that doesn't exist yet.

**Primitive Obsession** — using primitives (strings, ints) for things that are actually domain concepts with their own rules (an email address, money, a phone number), scattering validation and formatting logic at every use site instead of in one place.
→ **Replace Primitive with Object**: wrap the primitive in a small class that owns its own validation and behavior. A `Money` class that won't let you add USD to EUR by mistake is doing real work a raw `float` can't.

### Object-Orientation Abusers (misusing OO mechanisms)

**Switch Statements** (repeated, on the same type code, scattered across the codebase) — the same `switch (type)` logic duplicated wherever behavior needs to vary by type.
→ **Replace Conditional with Polymorphism**: give each type its own subclass implementing the varying behavior; call the method polymorphically instead of switching. See the `design-patterns` skill's Strategy entry for the composition-based alternative when subclassing doesn't fit.

**Refused Bequest** — a subclass that inherits methods it doesn't want or use, overriding them to throw or do nothing.
→ **Replace Inheritance with Delegation** (or re-examine the hierarchy): the subclass isn't really an "is-a" relationship to the parent; it's using inheritance for code reuse instead of genuine substitutability. Delegate to a held instance instead of inheriting.

### Change Preventers (structure that makes one conceptual change require touching many places)

**Divergent Change** — one class gets modified for many unrelated reasons (a change to billing logic and a change to reporting format both touch the same class).
→ **Extract Class**, splitting along which *reason to change* each piece serves — this is the Single Responsibility Principle made concrete: one class, one reason to change.

**Shotgun Surgery** — the opposite shape: one conceptual change requires touching many classes because the responsibility is scattered.
→ **Move Method / Move Field**: consolidate the scattered pieces into one place so future changes of that kind touch one file, not ten.

### Dispensables (things that shouldn't exist)

**Duplicated Code** — the same logic appearing in more than one place, guaranteed to drift out of sync the first time only one copy gets fixed.
→ **Extract Method/Function**, then call it from both sites. If the duplication is across sibling classes doing near-identical work with small variations, that's a Template Method or Strategy opportunity instead (see `design-patterns`).

**Dead Code** — code no longer called from anywhere.
→ **Delete it.** Version control remembers it if it's ever needed again; a comment claiming "keeping this in case we need it" is worse than deleting, because it makes the next reader wonder if something depends on it.

**Speculative Generality** — abstraction built for a variation that doesn't exist yet ("just in case we need multiple implementations someday").
→ **Collapse the abstraction** back to the concrete case. This is the direct violation of YAGNI (You Aren't Gonna Need It) — an interface with one implementation, a parameter that's always the same value, a hook no one calls, are all signs of this. Remove it; add it back if and when the second real case actually shows up.

### Couplers (excessive coupling between classes)

**Feature Envy** — a method that uses another object's data more than its own, reaching through getters repeatedly to do work that conceptually belongs on the other object.
→ **Move Method**: relocate the method to the class whose data it's actually working with. The tell: if a method's body is mostly `other.getX()`, `other.getY()`, `other.getZ()` combined together, that combination logic belongs inside `other`, not beside it.

**Inappropriate Intimacy** — two classes reaching into each other's internals (accessing private fields via reflection, or friend-class-style tight coupling), so a change to one routinely breaks the other.
→ **Move Method/Field** to consolidate the shared responsibility into one place, or **Extract Class** for the genuinely shared behavior so both original classes depend on it instead of on each other directly.

**Message Chains** — a call like `a.getB().getC().getD().doThing()`, coupling the caller to the entire structure of `A` through `D` just to reach one operation.
→ **Hide Delegate**: give `A` a method that internally does the chain-walk and returns what's needed, so callers depend on `A`'s interface, not on the whole object graph's shape.

## Automated vs. manual refactoring

Modern IDEs automate the mechanical, safe refactorings — rename, extract method/variable, inline, move — with guaranteed behavior preservation (the tool updates every reference correctly, something error-prone to do by hand with find-and-replace). Prefer the IDE's automated refactoring over a manual edit whenever one is available for the operation you're doing; reserve manual refactoring for judgment calls the tool can't make (which pieces belong together in a new class, which abstraction actually fits). The test suite is still the safety net for the judgment-call refactorings — the IDE's automation is the safety net for the mechanical ones.

## Using this catalog in review

When reviewing code and something feels off but you can't articulate why, run down this list — "feature envy" or "primitive obsession" is a more actionable review comment than "this feels messy," because it names both the problem and, via this catalog, the fix. This pairs with the `code-review` skill's Standards axis: a named smell from this catalog is a concrete, defensible finding, not a stylistic opinion.
