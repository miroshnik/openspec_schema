# This is openspec/specs/rate-limiter/spec.md after `openspec archive add-rate-limit` — a PURE
# native OpenSpec spec: a `## Purpose` + a `## Requirements` section, no spec-first frontmatter.
# The `## ADDED Requirements` delta rebuilt `## Requirements` verbatim. Purpose below is FILLED —
# the first archive stubs it "TBD … Update Purpose after archive" and filling it is OpenSpec's OWN
# native post-archive step. Coverage reads the scenarios here directly; the `rate-limiter` tests
# carry the matching `@spec` markers. The standard that requires this limiter lives, untouched by
# archive, at examples/add-rate-limit/standards/rate-limit-policy.md.

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
