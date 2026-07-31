# Review Notes

Every skill in this collection went through the same process before inclusion:

1. Confirm the source repository actually exists (not just a marketplace listing — a live `HEAD` request against GitHub)
2. Identify the real publisher and check for a plausible identity (a real org, a known individual, or at minimum a repo with genuine history)
3. Check what permissions/dependencies the skill declares, and whether they're justified by what the skill claims to do
4. Check for duplication against skills already in the collection
5. Check maintenance signals (stars, forks, rating, recency)

This mattered because marketplace badges turned out to be unreliable. Two concrete findings from this process:

- A skill listed as "Official / Verified" with a working-looking install command pointed to a GitHub repo that returned a 404 — the badge was simply wrong.
- A skill titled "Remotion Best Practices" was actually published by an unrelated aggregator account, not the real Remotion team, despite the name implying otherwise.

## Excluded as a class: one publisher's monorepo

Roughly 40 candidate skills came from a single publisher's monorepo. Individually, most looked fine. In aggregate, the pattern was hard to ignore:

- Nearly every skill in that catalog requested filesystem write, shell execution, *and* network access — including skills whose entire job was serving static reference text (a "Clean Code" guide, a "Database Architect" checklist) with no plausible reason to touch the network or a shell.
- Popularity stats (a specific star/fork count) were identical across dozens of unrelated skill listings — a sign of one inflated top-line number being reused, not per-skill signal.
- Real download counts were in the single digits for most individual skills, despite large "popularity" scores.
- The publisher-mislabeling incident above happened inside this same catalog.

No single one of these facts is disqualifying. Together, they were treated as a systemic trust signal and the whole catalog was excluded rather than cherry-picked, on the reasoning that an aggregate red flag outweighs any one skill's individual-looking safety.

## Excluded for other reasons

- **Dead/missing repositories**: several listings (a README generator, an Android design-guidelines skill, a "Startup Business Analyst" skill, a "Multiplayer Game Development" skill) had no resolvable source repo despite appearing as normal listings.
- **Abandoned**: a fork of a well-known LLM-council project with a single GitHub star and a required paid API key; a "Claude Scientific Skills" bundle with a confirmed 1.0/5 rating despite claiming 146+ sub-skills.
- **Financial-advice risk**: an equity-research skill and an options-trading-signal skill were excluded — both generate actionable financial recommendations from small, largely unaudited codebases with credibility-straining self-descriptions (one claimed a "165,000-line platform built solo in 9 days" backed by a 4-star, 1-fork repository).
- **Requires a third-party account or paid API key**: skills requiring Composio, Figma MCP, OpenRouter, Google Gemini API keys, or similar were held out rather than silently wired up, so nothing here surprises you with a bill or a new OAuth grant.
- **Real device/credential control**: a skill granting live Android device control via ADB, and a skill performing real Google-account browser automation with persistent plaintext session storage, were excluded — not because the code looked bad, but because that class of capability needs a human decision every time, not a blanket "yes" baked into a public skill collection.

## What this means for you

If a skill isn't in this repo, it might be genuinely bad — or it might just be something that needs your specific, informed yes (a paid API key, a device-control grant, a financial-advice tool) rather than a blanket include. Check the exclusion notes above before assuming "not here" means "not good."
