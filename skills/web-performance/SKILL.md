---
name: web-performance
description: Reference for web performance fundamentals — how HTTP/2 changes the rules for asset delivery, and reducing page weight and load time with performance budgets. Use this whenever the user is debugging slow page loads, deciding how to bundle/split assets, setting up a performance budget, or asking whether an old performance optimization technique (bundling, spriting, domain sharding) still applies under HTTP/2.
---

# Web Performance

Most page-load performance problems come from one of two places: sending more bytes than necessary, or sending them in a way that doesn't let the browser start using them as early as possible. Everything below is a variation on shrinking one of those two costs.

## HTTP/2 changed several long-standing "best practices" into anti-patterns

**The core HTTP/1.1 constraint that shaped a decade of practices**: browsers limit the number of simultaneous connections to one host (typically 6), and each connection can only have one request in flight at a time (head-of-line blocking) — so minimizing the *number* of requests mattered enormously, even at the cost of sending more total bytes or worse caching.

**What HTTP/2 changes**: multiplexing allows many requests and responses to interleave over a *single* connection simultaneously — the request-count penalty that justified several HTTP/1.1-era techniques mostly disappears.

**Bundling (concatenating many small files into one large one)** — under HTTP/1.1, this was close to mandatory (avoiding one request per file); under HTTP/2, it's often actively counterproductive. A single large bundle means the browser can't start executing/rendering anything until the whole bundle downloads, and a one-line change to any part of the bundle invalidates the browser cache for the *entire* bundle. Smaller, more granular files let the browser cache and reuse unchanged pieces independently, and HTTP/2's multiplexing means fetching many small files no longer carries the per-request penalty that made bundling worth this cost.

**Domain sharding (spreading assets across multiple hostnames to work around the per-host connection limit)** — actively harmful under HTTP/2: it defeats multiplexing (each additional hostname requires its own new connection, with its own connection-setup and TLS-handshake overhead) for a benefit (parallel connections) that HTTP/2 already provides for free over one connection. A site still sharding domains under HTTP/2 is paying real cost to work around a limitation that no longer exists.

**Spriting (combining many small images into one larger sprite sheet, fetched as one request)** — same logic as bundling: was worth it to avoid per-image request overhead under HTTP/1.1, mostly not worth it under HTTP/2, where individual images can be requested and cached independently without the old penalty.

**What still matters under HTTP/2**: total byte count sent still matters exactly as much as before — HTTP/2 changes the cost of *how many requests*, not the cost of *how many bytes*. Compression (gzip/brotli), image optimization, and removing genuinely unused code are all still fully worth doing; only the request-count-minimization techniques above lost their justification.

**Server push** (the HTTP/2 feature letting a server proactively send resources before the client requests them, anticipating what the page will need) — worth understanding but approach cautiously: it's easy to over-push resources the client already had cached, wasting bandwidth for no benefit, and in practice a well-configured `preload` resource hint (letting the *client* request early, rather than the server guessing what to push) has proven more reliable and is more widely supported going forward — check current browser support before relying on push specifically.

## Page weight and performance budgets

**Why "just optimize when it becomes a problem" doesn't work in practice**: page weight tends to grow gradually — one more tracking script here, one more unoptimized image there — with each individual addition looking small enough to not be worth blocking on. Nothing triggers a "let's fix performance now" moment until the cumulative effect is already bad, at which point the fix requires auditing and reversing many small additions instead of catching each one at the point it was added.

**A performance budget makes the cost visible at the point of decision, not after the fact.** Set explicit limits (total page weight, time-to-interactive, number of requests, whatever's most relevant to the site) and enforce them as part of the build/review process — a change that would push the page over budget gets flagged *at review time*, when it's cheap to reconsider, rather than discovered months later during a dedicated performance audit when the specific cause is buried among many other subsequent changes.

**What to include in a budget, roughly in order of typical impact**:
- **Total page weight** (all resources combined) — the blunt, easy-to-track top-level number.
- **JavaScript weight specifically** — usually the highest-leverage line item, since JS is the most expensive resource type per byte (it has to be parsed *and executed*, not just downloaded and rendered like an image).
- **Time-to-interactive** — when the page is actually usable, not just visually painted; a page that looks loaded but whose JS hasn't finished initializing yet is a common source of a frustrating "looks done, isn't responding" experience.
- **Largest Contentful Paint (LCP)** — when the largest above-the-fold content element finishes rendering, a good proxy for "does this feel fast to a real user," and one of the industry-standard Core Web Vitals metrics worth tracking specifically because it correlates well with actual user-perceived speed.

**Images are usually the single largest opportunity.** Serving appropriately-sized images (not a 4000px-wide image scaled down by CSS to 400px, which downloads all 4000px worth of data for a 400px display), modern compressed formats (WebP/AVIF over legacy JPEG/PNG where supported), and lazy-loading images below the fold (not fetched until the user scrolls near them) are each individually high-leverage and collectively often the biggest lever available on a typical content-heavy page.

**Measure with realistic conditions, not just a fast office connection.** A page that loads acceptably on a wired connection with a high-end machine can be substantially worse for real users on a mid-range mobile device over a throttled connection — testing tools that simulate slower CPU and network conditions give a far more representative picture of what most real users actually experience than testing exclusively under ideal development conditions.

## Practical checklist

- Is the site still using HTTP/1.1-era bundling, spriting, or domain sharding techniques after moving to HTTP/2, where they're now net-negative?
- Is there an explicit, enforced performance budget, or does page weight only get addressed reactively after it's already a problem?
- Are images served at appropriate sizes/formats, and lazy-loaded below the fold?
- Is performance measured under realistic (throttled, mobile-representative) conditions, not just a fast development connection?
- Is JavaScript weight tracked specifically, given its outsized cost relative to other resource types?
