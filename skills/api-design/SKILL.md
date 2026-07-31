---
name: api-design
description: Reference for designing web APIs that are predictable and pleasant to consume — resource naming, versioning strategy, pagination, and error response design. Use this whenever the user is designing a new REST/HTTP API, reviewing an API for consistency, deciding how to version or paginate an endpoint, or designing error response formats.
---

# API Design

A well-designed API is one a consumer can correctly guess the shape of before reading the docs, because it follows conventions consistently rather than inventing a new pattern per endpoint. Most of what makes an API "good" is this kind of predictability, not cleverness.

## Resource naming and structure

**Nouns for resources, HTTP methods for actions.** `/orders/123` plus `DELETE` is more consistent than `/deleteOrder/123` — the URL names the *thing*, the HTTP method names the *operation* on it. A URL with a verb baked in (`/getUser`, `/createOrder`) is usually a sign the API is modeling procedures instead of resources, which tends to produce an unpredictable, ad hoc endpoint list rather than a consistent pattern consumers can extrapolate from.

**Plural nouns, consistently.** `/orders` not `/order` for the collection, `/orders/123` for a single item — pick one convention and never mix singular and plural across endpoints; the inconsistency forces a consumer to memorize each endpoint individually rather than infer the pattern.

**Nest resources to express genuine ownership, but don't nest too deep.** `/orders/123/line-items` expresses that line items belong to a specific order — reasonable. `/customers/1/orders/123/line-items/5/discounts` is deep enough that it's fighting the URL structure to express what should probably be a flatter model with its own top-level resource and a filter, e.g., `/line-items/5/discounts?order=123`. A rough guide: if you need more than two levels of nesting to reach the resource you actually want, reconsider whether the deeper resource should be a top-level collection queryable by its parent's ID instead.

**Use HTTP status codes for what they actually mean, not loosely.** `200` for success, `201` specifically for a successful creation (with a `Location` header pointing at the new resource), `400` for a malformed request, `404` for a resource that doesn't exist, `422` for a well-formed request that fails validation, `500` for an unexpected server-side failure. Returning `200` with an error described only in the response body (a common shortcut) forces every consumer to parse the body just to know if the call succeeded, defeating the purpose of having status codes at all — clients (and infrastructure like load balancers, caches, and monitoring) can act correctly on a status code without parsing anything, but only if it's used honestly.

## Versioning strategy

**Why an API needs a versioning strategy from day one, not after the first breaking change is needed**: retrofitting versioning onto an API that's already been consumed unversioned means every existing consumer is implicitly "version 1" whether anyone planned for it or not — deciding the versioning scheme after the fact is strictly harder than deciding it up front, because there's no clean point to start from.

**Common approaches, with trade-offs**:
- **URL path versioning** (`/v1/orders`, `/v2/orders`) — the most visible and cache-friendly (different versions are trivially different URLs, so caching/routing infrastructure needs no special awareness of versioning at all), at the cost of the version living in every single endpoint's path.
- **Header versioning** (`Accept: application/vnd.api+json;version=2`) — keeps URLs stable across versions, but is less discoverable (a consumer has to know to look at headers, not just the URL) and harder to test casually in a browser or with a simple curl.
- **Query parameter versioning** (`/orders?version=2`) — simple to implement, but easy for a consumer to accidentally omit, silently falling back to a default version rather than getting an explicit error.

**Default recommendation**: URL path versioning for public APIs (maximizes discoverability and cacheability, which matter more for external consumers who won't have deep familiarity with the API's internals); header versioning is more defensible for internal APIs between services you control, where discoverability matters less and you can enforce header presence.

**What actually counts as a breaking change, worth bumping a major version for**: removing a field, renaming a field, changing a field's type or meaning, changing required-ness from optional to required, changing an endpoint's URL or method. What's *not* breaking, and shouldn't require a version bump: adding a new optional field to a response (a well-behaved consumer ignores unknown fields), adding a new endpoint, adding a new optional request parameter. Being disciplined about this distinction is what keeps version bumps rare and meaningful rather than happening on every deploy — and it's the same underlying discipline `build-systems-and-dependencies` covers for SemVer, applied to an API contract instead of a library.

## Pagination

**Why an unpaginated collection endpoint is a ticking problem, not a convenience**: a `/orders` endpoint that returns every order works fine with 50 orders and becomes a serious performance and reliability problem at 5 million — but by the time that's obviously true, consumers have already built against the unpaginated shape and adding pagination becomes a breaking change. Design pagination in from the start for any endpoint returning a collection, even when the current data volume makes it feel unnecessary.

**Offset-based pagination** (`?offset=100&limit=20`): simple to implement and understand, but has two real problems at scale — performance degrades for large offsets on many database implementations (skipping N rows still costs work proportional to N), and results can shift under a consumer paging through results if the underlying data changes between requests (an item inserted before the current offset shifts every subsequent page by one, causing skipped or duplicated items).

**Cursor-based pagination** (`?cursor=abc123&limit=20`, where the cursor encodes a stable position, typically derived from a unique, ordered field): avoids both offset problems — consistent performance regardless of how deep into the result set you are, and stable results even as underlying data changes, because the cursor anchors to a specific record rather than a numeric position that shifts. The cost is a less intuitive API (a consumer can't jump to "page 5" directly, only "the next page after this cursor") — usually a reasonable trade for any collection expected to grow large or change frequently while being paged through.

**Always include pagination metadata in the response** — total count (if computable without excessive cost), whether more pages exist, and the cursor/link for the next page — so a consumer doesn't have to infer pagination state from the shape of the results (e.g., guessing "fewer than `limit` results means this is the last page," which breaks the moment a filtered query happens to return exactly `limit` results on the last page).

## Error response design

**A consistent error shape across every endpoint**, not a different error format per endpoint or per failure type — a consumer should be able to write one error-handling code path that works everywhere in the API, not a special case per endpoint.

**A reasonable minimum error shape**:
```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The 'email' field is not a valid email address.",
    "details": [
      { "field": "email", "issue": "invalid_format" }
    ]
  }
}
```

- **A machine-readable `code`** the consumer's code can branch on reliably — never make the consumer parse the human-readable `message` string to figure out what kind of error occurred, since message text is free to change wording without warning and shouldn't be treated as an API contract.
- **A human-readable `message`** for logging and debugging, allowed to be more casual and to change wording over time without breaking anything, precisely because nothing should be parsing it programmatically.
- **Structured `details`** for validation-style errors where multiple things can be wrong at once (multiple invalid fields in one request) — a consumer building a form should be able to show every validation problem at once, not just the first one encountered, which requires the error format to support returning more than one issue.

**Don't leak internal implementation details into error messages** — a stack trace, an internal file path, or a raw database error message in an API response both looks unprofessional to a legitimate consumer and can hand an attacker useful information about internal structure. Log the internal detail server-side for debugging; return a sanitized, appropriately generic message to the client.

## Practical checklist

- Do resource URLs use nouns consistently, with HTTP methods carrying the verb?
- Are status codes used for what they actually mean, not just 200-with-an-error-in-the-body?
- Is there an explicit versioning strategy, decided before the first breaking change is needed rather than after?
- Does every collection endpoint paginate, even ones with small data today?
- Is the error response shape identical across every endpoint, with a machine-readable code separate from the human-readable message?
