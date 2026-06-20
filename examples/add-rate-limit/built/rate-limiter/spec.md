---
# Frontmatter CORE — re-applied AFTER first archive (the CREATE archive dropped it). This is
# what openspec/specs/rate-limiter/spec.md looks like once re-applied. Functional capability →
# no `governs:`. CORE is test: + tier: only; the `### Requirement:` SHALL/MUST clause below is
# the authoritative one-liner (no separate invariant field). The gate's re-apply sub-check fails
# this built spec if test: is missing, so a forgotten re-apply cannot silently pass coverage.
test:       test/rate-limiter/withRateLimit.test.ts   # each test case tagged `@spec rate-limiter/<requirement>/<scenario>` (one marker per scenario). Code under test: src/middleware/withRateLimit.ts (named here + in prose, not a frontmatter field).
tier:       mechanical
# adoption: omitted — active spec.
---

# Rate limiter

## Purpose

Bound how fast any one client can consume the public API, so a single caller cannot
exhaust upstream capacity or run up unbounded cost. The limiter is the mechanism the
`rate-limit-policy` standard requires every public handler to be wrapped by.

## Requirements

### Requirement: Public API requests are rate-limited per client

The public HTTP API SHALL admit each client request only when that client's token
bucket has capacity, and MUST reject an over-limit request with HTTP 429 and a
`Retry-After` header stating when capacity returns. The limiter MUST key buckets by
the authenticated client id (falling back to source IP for anonymous traffic) and MUST
decide before the handler runs, so no business logic executes for a rejected request.

#### Scenario: under limit — request is allowed

- **Given** a client whose token bucket has remaining capacity
- **When** the client sends a request to a public endpoint
- **Then** the limiter consumes one token and the handler runs, returning its normal
  response with an `X-RateLimit-Remaining` header reflecting the decremented count.

#### Scenario: over limit — request is rejected with 429 + Retry-After

- **Given** a client whose token bucket is exhausted within the current window
- **When** the client sends another request to a public endpoint
- **Then** the limiter short-circuits before the handler, responding `429 Too Many
  Requests` with a `Retry-After` header (seconds until a token refills) and no
  side effects from the handler.
