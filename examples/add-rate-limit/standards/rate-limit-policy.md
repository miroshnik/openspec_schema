---
# A STANDARD is a SELF-CONTAINED living record at <repo>/standards/<name>.md — TOP-LEVEL,
# OUTSIDE openspec/, NOT an OpenSpec spec. It is a committed git file `openspec archive` NEVER
# touches, so there is no delta and no built form. Frontmatter is SAFE here (the file is never
# rebuilt). `governs:` scopes the rule; `tier:` is machine-checkable (mechanical) vs
# needs-human-judgment (judge) — NOT the mechanism. The gate (not `openspec validate`) validates
# this file: governs + tier present; a `### Requirement:` with SHALL/MUST + a `conforms` scenario
# + a `violates` scenario. RETRIEVE reads standards/ directly; the gate reads the scenarios below
# for coverage. The enforcer (eslint-plugin-local/require-rate-limit) is NAMED in the rule prose +
# the check-test + checks/rate-limit-policy.check.md, NOT a frontmatter field.
governs:    public HTTP handlers — every export mounted on the public router (src/routes/public/**)
tier:       mechanical
---

# Rate-limit policy

A public API with no uniform limiter is a denial-of-service and cost-blowout surface, and
ad-hoc per-handler limits drift and leave gaps as endpoints are added. This standard makes
"every public handler is rate-limited" a checked, boundary-level rule so a new endpoint cannot
ship unlimited by omission. Everything about the standard — its scope, tier, why, the rule, its
scenarios, and its amendment history — lives in THIS file, always visible.

## Decision Record

- **Status:** accepted
- **Context:** the public API exposes many handlers; without a uniform limiter any one client
  can exhaust upstream capacity and run up cost. Per-handler manual limits drift and new
  endpoints are added unlimited by oversight. The design standards loop hit ungoverned ground
  (RETRIEVE found nothing scoping the public router for limiting) → CREATE.
- **Decision:** enforce rate limiting at the ROUTER BOUNDARY — every export on the public router
  MUST be wrapped by `withRateLimit` — verified by an AST rule, not reviewer vigilance.
- **Consequences:** every public handler is uniformly limited; a new unwrapped endpoint fails the
  check (fails closed). Cost: one wrapper per export and one shared limiter to tune.
- **Alternatives considered:** per-handler manual limits → drift / coverage gaps; gateway-only
  limiting → no per-handler keying and invisible to the in-repo gate.

## Rule

### Requirement: Public handlers must be rate-limited

Every export mounted on the public router SHALL be wrapped by `withRateLimit` before it is
served, and an unwrapped public handler MUST be rejected by the check. The rule binds at the
router boundary (`src/routes/public/**`), not inside individual handlers, so a new endpoint
cannot ship unlimited by omission.

#### Scenario: conforms

- **Given** a public route file that exports `withRateLimit(handler)` for every route
- **When** the `require-rate-limit` AST check runs
- **Then** it passes.

#### Scenario: violates

- **Given** a public route file exporting a bare `handler` not wrapped by `withRateLimit`
- **When** the `require-rate-limit` AST check runs
- **Then** it fails, naming the file and export and pointing the author at `withRateLimit`.

## Rationale log

<!-- The standard's amendment history (the ADR's living tail). Accretes; never rewrite history. -->
- **(2026-06-20):** boundary-level AST enforcement chosen over a runtime assertion so an
  unlimited handler fails at check time, before it can ever serve traffic. Keyed at the router so
  the rule is a pure structural fact (export ⊂ public router ⇒ must be wrapped), which is why
  `tier: mechanical` suffices with no judge clause.
