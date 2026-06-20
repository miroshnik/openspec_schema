# Tasks — add-rate-limit

SOURCE-FIRST (#0): the specs under `change/specs/` are authoritative — and the SCENARIO is the
traceable, generative unit. Every item below is a PROJECTION of a scenario, regenerable from it:
limiter code projects the `rate-limiter` scenarios; the policy enforcer projects the
`rate-limit-policy` conforms/violates scenarios; the codemod projects the standard's tightening;
each test projects the scenario(s) it carries a `@spec` marker for. To change behavior, amend the
spec/scenario FIRST, then RE-PROJECT its test — never edit a derivative behind a scenario you did
not touch.

## 1. Functional capability — the limiter

- [ ] Author `src/middleware/withRateLimit.ts` — token-bucket wrapper, keyed by client id
      (IP fallback), deciding before the handler runs.
- [ ] Implement under-limit path: consume a token, run handler, set `X-RateLimit-Remaining`.
- [ ] Implement over-limit path: short-circuit with `429` + `Retry-After` (seconds to refill).
- [ ] Author `test/rate-limiter/withRateLimit.test.ts` realizing BOTH scenarios:
      `under limit → allowed`, `over limit → 429 + Retry-After`. (Behavioral scenarios → an
      EXECUTABLE test; the projection FORM is the stack's choice — unit/e2e/property/contract/
      type-level/runtime-assertion — not necessarily lint.)
- [ ] Add a traceability marker PER SCENARIO, on each test case (the `it()`/`test` block that
      projects it) — one test may carry several:
      `// @spec rate-limiter/Public API requests are rate-limited per client/under limit — request is allowed`
      on the under-limit case, and
      `// @spec rate-limiter/Public API requests are rate-limited per client/over limit — request is rejected with 429 + Retry-After`
      on the over-limit case, so coverage sees EACH scenario projected.

## 2. Standard — the policy + its AST check

- [ ] Author the `eslint-plugin-local/require-rate-limit` AST rule
      (`src/lib/eslint-rules/require-rate-limit.ts`): flag any export in `src/routes/public/**`
      not wrapped by `withRateLimit`. A lint/AST rule is ONE projection form, chosen HERE because
      this scenario is a structural AST fact (export ⊂ public router ⇒ wrapped); a behavioral
      scenario projects to an executable test instead. The form is the stack's choice. The rule id
      lives in `checks/rate-limit-policy.check.md` (the rule spec + fixtures) and is named in the
      standard's prose — NOT a frontmatter field.
- [ ] Wire the rule into eslint config for `src/routes/public/**`.
- [ ] Author `test/standards/rate-limit-policy.test.ts` — the CHECK-TEST: assert the rule
      PASSES the `conforms` fixture (wrapped handler) and FLAGS the `violates` fixture
      (bare handler). These fixtures ARE the standard's scenarios (the RuleTester `valid` group
      projects `conforms`, the `invalid` group projects `violates`).
- [ ] Add the scenario marker to EACH fixture group — TWO markers on this one check-test:
      `// @spec rate-limit-policy/Public handlers must be rate-limited/conforms`
      on the `valid` group and
      `// @spec rate-limit-policy/Public handlers must be rate-limited/violates`
      on the `invalid` group, so coverage sees BOTH scenarios projected.

## 3. Migration — bring existing handlers into conformance

- [ ] Run/author the codemod wrapping the **3 existing unwrapped public handlers** on
      `src/routes/public/**` with `withRateLimit` (projection of the standard's tightening).
- [ ] Verify each wrapped handler still passes its existing functional tests (no behavior
      change beyond limiting).

## 4. Gate (LAST group — must be GREEN before the change is complete)

- [ ] Run `npm run gate` after EACH group above (continuously), and as the final group.
      It aggregates, failing on first red:
      1. project mechanical checks (full eslint + vitest suite — regression guard; the
         `eslint-plugin-local/require-rate-limit` rule runs here in `eslint .`);
      2. org-standards integrity (none pinned — skipped);
      3. coverage — per touched spec, EVERY `#### Scenario:` must have ≥1 test carrying its
         `@spec <capability>/<requirement>/<scenario>` marker (the CORE keys on `test:` only;
         there is no `check:` to resolve). A declared scenario with no projecting test ⇒
         declared-but-unprojected ⇒ FAIL — stricter than the old requirement-level linkage, and
         the faithful operationalization of "tests are projections of scenarios." Here that means
         all 4 scenarios (2 limiter + conforms/violates) must each be marked. The gate also
         VERIFIES each declared `tier:` matches what `test:` resolves to (both here resolve to
         mechanical executable artifacts ⇒ `mechanical`) and fails on mismatch;
      4. regression diff vs merge-base (both specs are NEW → no baseline; coverage governs);
      5. `openspec validate --strict`;
      6. judge (tier-judge clauses) — none here; soft in CI regardless.
- [ ] Confirm the gate is GREEN. (Before this group it is RED — see README: the new standard
      has no test yet AND the AST check flags the 3 existing unlimited handlers.)

## 5. Archive

- [ ] `openspec archive add-rate-limit` — merges both deltas into `openspec/specs/`.

## 6. POST-ARCHIVE re-apply (new specs — REQUIRED on OpenSpec 1.4.1; backstopped by the gate)

- [ ] Both `rate-limiter` and `rate-limit-policy` are NEW, so their FIRST archive rebuilt the
      main spec from the ADDED delta ALONE — dropping frontmatter CORE (and, for the standard,
      the ADR + Rationale log) and stubbing `## Purpose` "TBD". RE-APPLY:
  - [ ] Re-add frontmatter CORE (`test:` + `tier:`) + fill `## Purpose` in
        `openspec/specs/rate-limiter/spec.md`
        (model: `examples/add-rate-limit/built/rate-limiter/spec.md`).
  - [ ] Re-add frontmatter CORE (`test:` + `tier:` + `governs:`) + `## Decision Record` (above
        `## Requirements`) + trailing `## Rationale log` + fill `## Purpose` in
        `openspec/specs/rate-limit-policy/spec.md`
        (model: `examples/add-rate-limit/built/rate-limit-policy/spec.md`).
- [ ] This re-apply is BACKSTOPPED by the gate, not an honor-system chore: after a new spec's
      CREATE archive the gate's coverage sub-check FAILS if the built spec is missing its CORE
      (`test:`), so a forgotten re-apply cannot silently pass. RE-RUN `npm run gate` over the
      updated `openspec/specs/` — without the re-applied CORE, coverage has no `test:` to read.
      Must be GREEN.
