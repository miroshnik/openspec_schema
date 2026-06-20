---
# STANDARD = non-functional, cross-cutting capability. The `governs:` field is what makes
# it a standard rather than a functional spec. Frontmatter CORE above the delta tokens so
# archive preserves it. tier: mechanical → an AST/lint rule is the hard enforcer. CORE keys
# the gate's coverage on test: only; the enforcer (eslint-plugin-local/require-rate-limit)
# is NAMED in the test body + the spec prose + checks/rate-limit-policy.check.md, NOT here.
test:       test/standards/rate-limit-policy.test.ts # MANDATORY — passes the `conforms` fixture, flags the `violates` fixture; each fixture group tagged `@spec rate-limit-policy/<requirement>/<scenario>` (one marker per scenario, two here). Runs the `eslint-plugin-local/require-rate-limit` AST rule (one projection form, chosen because this scenario is structural; see checks/rate-limit-policy.check.md).
tier:       mechanical
governs:    public HTTP handlers — every export mounted on the public router (src/routes/public/**)
# adoption: omitted — active/new standard. (Presence of `adoption: legacy` would exempt a
#           pre-existing spec from coverage until first modified.)
---

# DELTA for change/add-rate-limit — kept MINIMAL on purpose. The ADR is DECIDED in design.md and
# the full Decision Record + Rationale log live in the BUILT spec; neither is placed in this delta
# body. Why: on a standard's FIRST archive (CREATE) OpenSpec builds the main spec from the ADDED
# delta body VERBATIM while dropping the frontmatter CORE and stubbing `## Purpose` "TBD" — so any
# prose left inside `## ADDED Requirements` would survive as cruft inside the built `## Requirements`.
# We therefore author ONLY the requirement + scenarios here, then RE-APPLY the frontmatter CORE +
# `## Decision Record` + `## Rationale log` to openspec/specs/rate-limit-policy/spec.md after archive
# (tasks.md group 6; the re-applied form is examples/add-rate-limit/built/rate-limit-policy/spec.md).

## ADDED Requirements

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
