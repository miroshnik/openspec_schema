# Design — add-rate-limit

## Context

The public API mounts handlers on `src/routes/public/**`. Today none are rate-limited.
We need both the behavior (a limiter) and a durable rule (a standard) so the gap cannot
silently reopen. Per Principle #0 the spec/standard is the source of truth; this design
DECIDES — the functional spec is AUTHORED in `specs/`, and the standard's living record is
AUTHORED at `standards/rate-limit-policy.md` (a top-level file OUTSIDE the change folder and
outside openspec/, so archive never touches it). Recording the full decision in design would
invert #0, putting the recorded decision before its source.

## Standards loop (spec-first)

**RETRIEVE** — read `standards/` directly and pull the records whose `governs:` covers the
affected surfaces from the proposal (`src/routes/public/**`, `src/middleware/**`, eslint rules).
Result: **nothing governs rate limiting yet.** No existing standard scopes the public router for
limiting. This is ungoverned ground, not a comply case.

For the one cross-cutting decision (should public handlers be uniformly rate-limited, and how is
that enforced?):

- **CREATE** — `rate-limit-policy`, authored as the self-contained record
  `standards/rate-limit-policy.md`. This is ungoverned ground AND an architectural call (enforce
  at the router boundary vs per-handler), so the record carries a `## Decision Record` (ADR).
  - **rule (the SHALL clause):** no public-router export serves traffic unwrapped by
    `withRateLimit`. This is the standard's `### Requirement:` one-liner under `## Rule`, not a
    separate frontmatter field.
  - **tier:** `mechanical` — boundary membership is a pure AST fact; mechanize first, no judge
    tier needed for the structural rule. `tier:` means machine-checkable-at-all, NOT the
    mechanism; the scenario's NATURE only hints the projection CLASS (this one is structural →
    static analysis). The gate VERIFIES this tier against what the standard's check-test resolves
    to (a machine-checkable artifact ⇒ mechanical) and fails on mismatch.
  - **enforcer + check-test:** RESOLVE the enforcer first. Searched existing project checks
    (eslint config, tsconfig, CI) — none enforces wrapper-at-boundary. The projection FORM is
    STACK-SPECIFIC (any of unit/e2e/property/contract/type-level/runtime-assertion/lint-AST —
    whatever this stack uses to check the scenario); a lint/AST rule is the right form HERE only
    because this scenario is a structural AST fact, whereas the limiter's behavioral scenarios
    project to an executable test (below). So **AUTHOR** an AST rule
    `eslint-plugin-local/require-rate-limit` (every export in the public router is wrapped by
    `withRateLimit`), then **AUTHOR** the check-test asserting the rule FLAGS an unwrapped handler
    fixture and PASSES a wrapped one — one `@spec` marker PER SCENARIO, so the `conforms` and
    `violates` scenarios each project (their fixtures ARE the scenarios). The rule id is named in
    the standard's `## Rule` prose + the check-test + `checks/rate-limit-policy.check.md`, not a
    frontmatter field.
  - **the living record:** authored at `standards/rate-limit-policy.md` — scope, tier, the
    Decision Record, the rule, its conforms/violates scenarios, and the Rationale log all live
    THERE, always visible. The gate validates it (governs + tier present; a SHALL/MUST rule with a
    conforms scenario and a violates scenario) and reads its scenarios for coverage. Because the
    file is a committed git file archive never touches, its frontmatter is safe. Bias to FEW: this
    is the only standard this change adds.

- **DEFER (not taken here, but considered):** bias-to-FEW asks whether this is too early to
  standardize — one surface, not clearly a class yet. If so, KEEP it local as `rate-limiter`'s own
  behavior and record it only as a standard CANDIDATE (a line here), promoting it via CREATE + a
  migration when a second surface needs the rule or a reviewer flags it. We chose CREATE, not
  DEFER: the rule must govern a CLASS — every export on the public router, including endpoints not
  yet written — so it has already outgrown one capability. You do not standardize on a hunch; here
  the class is concrete.

**RATIFICATION** — this CREATE is surfaced for human sign-off at PR review (the soft-CI judge
posts a "standards-changed" suspect; a human approves). We do not self-merge a standards change.

## The limiter (functional capability)

`rate-limiter` is a functional capability, NOT a standard (no `governs:`, and it lives as a pure
native OpenSpec spec under `specs/`). It provides the `withRateLimit(handler)` middleware:

- **Algorithm:** token bucket, keyed by authenticated client id (fallback: source IP).
- **Decision before handler:** the wrapper decides admit/reject before invoking the inner
  handler, so a rejected request has no side effects.
- **Over-limit response:** `429 Too Many Requests` + `Retry-After` (seconds to refill).
- **Bucket store:** pluggable; in-memory for dev/test, Redis in prod. Ephemeral — no data
  migration on rollback.

Its **test is a projection of its scenarios** — `under limit → allow` and `over limit → 429 +
Retry-After` are BEHAVIORAL, so they project to an EXECUTABLE test (`test/rate-limiter/
withRateLimit.test.ts`), each scenario a case carrying its own `@spec` marker. We do not invent a
test contract; the scenarios ARE the contract, and the test is their regenerable, stack-local
artifact — re-project it when a scenario changes.

## Enforcement decisions (recorded; mechanics live in tasks)

The functional spec carries no frontmatter; the standard's record carries `governs:` + `tier:`
and names its enforcer in prose, not a field. There is no `check:` field anywhere. The enforcer
(the code under test, or a standard's check in whatever FORM the stack uses —
unit/e2e/property/contract/type-level/runtime-assertion/lint-AST) is RESOLVED-or-AUTHORED here,
named in the test body + the record's `## Rule` prose, and PROVEN by its check-test. The form is
stack-specific and follows the scenario's nature (behavioral → executable test; structural →
static analysis), not a fixed mechanism. The gate verifies each declared `tier:` matches what its
check-test resolves to.

| spec / standard | SHALL one-liner (the `### Requirement:`) | enforcer (named in test/prose) | projection form (stack-specific) | reuse vs author | the coverage key |
| --- | --- | --- | --- | --- | --- |
| `rate-limiter` (functional spec) | every public request is admitted only under a token-bucket decision (else 429 + Retry-After) | `withRateLimit` middleware | executable test (behavioral scenarios) | AUTHOR (new code) | AUTHOR scenario tests (one `@spec` marker per scenario) |
| `rate-limit-policy` (standard record) | every public-router export is wrapped by `withRateLimit` | `eslint-plugin-local/require-rate-limit` | lint/AST rule (structural scenario) | AUTHOR (none existed) | AUTHOR check-test (one marker per scenario: passes conforms / flags violates) |

## Migration

Adding the standard makes the 3 existing unwrapped public handlers immediately non-conforming —
the AST check will flag them the moment it lands. A **codemod** wraps each export on
`src/routes/public/**` with `withRateLimit`. This migration is itself a projection of the change
(the standard tightened the contract; the codemod brings existing code into conformance).
Mechanics and the gate sequencing are in `tasks.md`.
