---
# FUNCTIONAL capability — a component's behavior. NO `governs:` (that field is what
# marks a STANDARD). Frontmatter CORE lives here so archive preserves it (above the
# delta tokens / ## Requirements) and so the gate's coverage can read it regardless of
# body shape. CORE is just test: + tier: here — the authoritative one-liner IS the
# `### Requirement:` SHALL/MUST clause below; there is no separate invariant field.
test:       test/rate-limiter/withRateLimit.test.ts  # MANDATORY — projects the scenarios below; each test case tagged `@spec rate-limiter/<requirement>/<scenario>` (one marker per scenario). The limiter (src/middleware/withRateLimit.ts) is the code under test, named in the test + prose, not a frontmatter field.
tier:       mechanical
# governs: absent on purpose — this is a functional capability, not a standard.
# adoption: omitted — this spec is active (presence of `adoption: legacy` would exempt a
#           pre-existing spec from coverage until first modified; this one is new).
---

# This is a DELTA, authored under change/add-rate-limit. It is NOT the built main spec.
# Tokens are EXACT and load-bearing: `## ADDED Requirements` (level-2), `### Requirement:`
# (level-3), `#### Scenario:` (level-4, exactly four hashes), literal SHALL/MUST in the body.
# On FIRST archive this delta ALONE rebuilds openspec/specs/rate-limiter/spec.md — see
# examples/add-rate-limit/built/rate-limiter/spec.md for the re-applied result.

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
