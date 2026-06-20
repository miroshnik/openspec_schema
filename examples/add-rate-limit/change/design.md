# Design — add-rate-limit

## Context

The public API mounts handlers on `src/routes/public/**`. Today none are rate-limited.
We need both the behavior (a limiter) and a durable rule (a standard) so the gap cannot
silently reopen. Per Principle #0 the specs are the source of truth; this design only
DECIDES — the standard itself is AUTHORED in `specs/` (writing it here would invert #0,
putting the recorded decision before its source spec).

## Standards loop (spec-first)

**RETRIEVE** — pull standards whose `governs:` covers the affected surfaces from the
proposal (`src/routes/public/**`, `src/middleware/**`, eslint rules). Result: **nothing
governs rate limiting yet.** No existing standard scopes the public router for limiting.
This is ungoverned ground, not a comply case.

For the one cross-cutting decision (should public handlers be uniformly rate-limited, and
how is that enforced?):

- **CREATE** — `rate-limit-policy`. This is ungoverned ground AND an architectural call
  (enforce at the router boundary vs per-handler), so it needs a Decision Record.
  - **rule (the SHALL clause):** no public-router export serves traffic unwrapped by
    `withRateLimit`. This is the standard's `### Requirement:` one-liner, not a separate
    frontmatter field.
  - **tier:** `mechanical` — boundary membership is a pure AST fact; mechanize first, no
    judge tier needed for the structural rule. `tier:` means machine-checkable-at-all, NOT the
    mechanism; the scenario's NATURE only hints the projection CLASS (this one is structural →
    static analysis). The gate VERIFIES this tier against what `test:` resolves to (a
    machine-checkable artifact ⇒ mechanical) and fails on mismatch.
  - **enforcer + test (fold into `test:`):** RESOLVE the enforcer first. Searched existing
    project checks (eslint config, tsconfig, CI) — none enforces wrapper-at-boundary. The
    projection FORM is STACK-SPECIFIC (any of unit/e2e/property/contract/type-level/
    runtime-assertion/lint-AST — whatever this stack uses to check the scenario); a lint/AST
    rule is the right form HERE only because this scenario is a structural AST fact, whereas
    the limiter's behavioral scenarios project to an executable test (below). So **AUTHOR** an
    AST rule `eslint-plugin-local/require-rate-limit` (every export in the public router is
    wrapped by `withRateLimit`), then **AUTHOR** the check-test asserting the rule FLAGS an
    unwrapped handler fixture and PASSES a wrapped one — one marker PER SCENARIO, so the
    `conforms` and `violates` scenarios each project (their fixtures ARE the scenarios). The
    standard's CORE carries only `test:` — that test RUNS the AST rule; the rule id is named in
    the test body + spec prose + `checks/rate-limit-policy.check.md`, not a frontmatter field.
  - **Decision Record:** authored in the standard's spec
    (`specs/rate-limit-policy/spec.md`), not here — design records the call; specs is its
    authoritative home. Bias to FEW: this is the only standard this change adds.

**RATIFICATION** — this CREATE is surfaced for human sign-off at PR review (the soft-CI
judge posts a "standards-changed" suspect; a human approves). We do not self-merge a
standards change.

## The limiter (functional capability)

`rate-limiter` is a functional capability, NOT a standard (no `governs:`). It provides the
`withRateLimit(handler)` middleware:

- **Algorithm:** token bucket, keyed by authenticated client id (fallback: source IP).
- **Decision before handler:** the wrapper decides admit/reject before invoking the inner
  handler, so a rejected request has no side effects.
- **Over-limit response:** `429 Too Many Requests` + `Retry-After` (seconds to refill).
- **Bucket store:** pluggable; in-memory for dev/test, Redis in prod. Ephemeral — no data
  migration on rollback.

Its **test is a projection of its scenarios** — `under limit → allow` and `over limit → 429 +
Retry-After` are BEHAVIORAL, so they project to an EXECUTABLE test (`test/rate-limiter/
withRateLimit.test.ts`), each scenario a case carrying its own `@spec` marker. We do not invent
a test contract; the scenarios ARE the contract, and the test is their regenerable, stack-local
artifact — re-project it when a scenario changes.

## Enforcement decisions (recorded; mechanics live in tasks)

The CORE keys on `test:` only — there is no `check:` field. The enforcer (the code under
test, or a standard's check in whatever FORM the stack uses — unit/e2e/property/contract/
type-level/runtime-assertion/lint-AST) is RESOLVED-or-AUTHORED here, named in the test body +
spec prose, and PROVEN by that `test:`. The form is stack-specific and follows the scenario's
nature (behavioral → executable test; structural → static analysis), not a fixed mechanism. The
gate verifies each declared `tier:` matches what its `test:` resolves to.

| spec | SHALL one-liner (the `### Requirement:`) | enforcer (named in test/prose) | projection form (stack-specific) | reuse vs author | test: (the coverage key) |
| --- | --- | --- | --- | --- | --- |
| `rate-limiter` | every public request is admitted only under a token-bucket decision (else 429 + Retry-After) | `withRateLimit` middleware | executable test (behavioral scenarios) | AUTHOR (new code) | AUTHOR scenario tests (one `@spec` marker per scenario) |
| `rate-limit-policy` | every public-router export is wrapped by `withRateLimit` | `eslint-plugin-local/require-rate-limit` | lint/AST rule (structural scenario) | AUTHOR (none existed) | AUTHOR check-test (one marker per scenario: passes conforms / flags violates) |

## Migration

Adding the standard makes the 3 existing unwrapped public handlers immediately
non-conforming — the AST check will flag them the moment it lands. A **codemod** wraps
each export on `src/routes/public/**` with `withRateLimit`. This migration is itself a
projection of the change (the standard tightened the contract; the codemod brings existing
code into conformance). Mechanics and the gate sequencing are in `tasks.md`.
