# FUNCTIONAL capability — a PURE native OpenSpec spec delta. NO spec-first frontmatter at all. A
# functional capability survives archive NATIVELY: the `## ADDED Requirements` delta ALONE rebuilds
# openspec/specs/rate-limiter/spec.md.
#
# This is a DELTA, authored under changes/add-rate-limit. It is NOT the built main spec. Tokens
# are EXACT and load-bearing: `## ADDED Requirements` (level-2), `### Requirement:` (level-3),
# `#### Scenario:` (level-4, exactly four hashes), literal SHALL/MUST in the body. The scenarios
# below ARE the contract; the limiter's tests are their stack-local projection (one `@spec`
# marker per scenario). See examples/add-rate-limit/built/rate-limiter/spec.md for the built form.
#
# `## Purpose`: a brand-new spec's first archive stubs Purpose "TBD … Update Purpose after
# archive" — filling it is OpenSpec's OWN native step on the built spec. (The built example shows
# it filled.)

## ADDED Requirements

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
