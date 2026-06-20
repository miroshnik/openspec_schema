---
# Frontmatter CORE — re-applied AFTER first archive. The CREATE archive rebuilt this file
# from the ADDED delta alone and DROPPED this block, the Decision Record, and the Rationale
# log, stubbing Purpose "TBD". This file shows openspec/specs/rate-limit-policy/spec.md after
# the re-apply task. STANDARD → carries `governs:`. CORE keys coverage on test: only; the
# enforcer (eslint-plugin-local/require-rate-limit) is NAMED in the test + prose, not here.
# The gate's re-apply sub-check fails this built spec if test: is missing.
test:       test/standards/rate-limit-policy.test.ts   # each fixture group tagged `@spec rate-limit-policy/<requirement>/<scenario>` (one marker per scenario, two here). Runs the `eslint-plugin-local/require-rate-limit` AST rule — one projection form, chosen because this scenario is structural (see checks/rate-limit-policy.check.md).
tier:       mechanical
governs:    public HTTP handlers — every export mounted on the public router (src/routes/public/**)
# adoption: omitted — active standard.
---

# Rate-limit policy

## Purpose

A public API with no uniform limiter is a denial-of-service and cost-blowout surface, and
ad-hoc per-handler limits drift and leave gaps as endpoints are added. This standard makes
"every public handler is rate-limited" a checked, boundary-level rule so a new endpoint
cannot ship unlimited by omission.

## Decision Record

<!-- ADR — above ## Requirements so archive preserves it on later MODIFY archives. -->
- **Status:** accepted
- **Context:** the public API exposes many handlers; without a uniform limiter any one client
  can exhaust upstream capacity and run up cost. Per-handler manual limits drift and new
  endpoints are added unlimited by oversight.
- **Decision:** enforce rate limiting at the ROUTER BOUNDARY — every export on the public
  router MUST be wrapped by `withRateLimit` — verified by an AST rule, not reviewer vigilance.
- **Consequences:** every public handler is uniformly limited; a new unwrapped endpoint fails
  the check (fails closed). Cost: one wrapper per export and one shared limiter to tune.
- **Alternatives considered:** per-handler manual limits → drift / coverage gaps; gateway-only
  limiting → no per-handler keying and invisible to the in-repo gate.

## Requirements

### Requirement: Public handlers must be rate-limited

Every export mounted on the public router SHALL be wrapped by `withRateLimit` before it
is served, and an unwrapped public handler MUST be rejected by the check. The rule binds
at the router boundary (`src/routes/public/**`), not inside individual handlers, so a new
endpoint cannot ship unlimited by omission.

#### Scenario: conforms

- **Given** a public route file that exports `withRateLimit(handler)` for every route
- **When** the `require-rate-limit` AST check runs
- **Then** it passes.

#### Scenario: violates

- **Given** a public route file exporting a bare `handler` not wrapped by `withRateLimit`
- **When** the `require-rate-limit` AST check runs
- **Then** it fails, naming the file and export and pointing the author at `withRateLimit`.

## Rationale log

<!-- Trailing section — survives archive. Accretes; never rewrite history. -->
- **v1 (2026-06-20):** boundary-level AST enforcement chosen over a runtime assertion so an
  unlimited handler fails at check time, before it can ever serve traffic. Keyed at the router
  so the rule is a pure structural fact (export ⊂ public router ⇒ must be wrapped), which is why
  `tier: mechanical` suffices with no judge clause.
