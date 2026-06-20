# Tasks — add-rate-limit

SOURCE-FIRST (#0): the `rate-limiter` spec under `change/specs/` and the `rate-limit-policy`
record under `standards/` are authoritative — and the SCENARIO is the traceable, generative unit.
Every item below is a PROJECTION of a scenario, regenerable from it: limiter code projects the
`rate-limiter` scenarios; the policy enforcer projects the `rate-limit-policy` conforms/violates
scenarios; the codemod projects the standard's tightening; each test projects the scenario(s) it
carries a `@spec` marker for. To change behavior, amend the spec/scenario FIRST, then RE-PROJECT
its test — never edit a derivative behind a scenario you did not touch.

Everything below happens in ONE pass, pre-archive: author the functional spec + the standard
record + tests + the check + the migration, then run the gate green. The gate reads the CHANGE
(the `## ADDED Requirements` delta + the `standards/` record + the test markers + the full suite +
git) — everything it needs exists before archive.

## 1. Functional capability — the limiter (spec + tests)

- [ ] Confirm `change/specs/rate-limiter/spec.md` is a pure native delta (`## ADDED Requirements`
      + `### Requirement:` SHALL/MUST + the two `#### Scenario:` blocks). No frontmatter.
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

## 2. Standard — the policy record + its AST check-test

- [ ] Confirm the living record `standards/rate-limit-policy.md` is complete: frontmatter
      (`governs:` + `tier: mechanical`); `## Decision Record` (ADR); `## Rule` with the
      `### Requirement:` SHALL/MUST clause and the `conforms` + `violates` `#### Scenario:` blocks;
      `## Rationale log`. (The standard is NOT an OpenSpec spec — no delta, no built form;
      `openspec validate` does not touch it, the gate validates it.)
- [ ] Author the `eslint-plugin-local/require-rate-limit` AST rule
      (`src/lib/eslint-rules/require-rate-limit.ts`): flag any export in `src/routes/public/**`
      not wrapped by `withRateLimit`. A lint/AST rule is ONE projection form, chosen HERE because
      this scenario is a structural AST fact (export ⊂ public router ⇒ wrapped); a behavioral
      scenario projects to an executable test instead. The form is the stack's choice. The rule id
      lives in `checks/rate-limit-policy.check.md` (the rule spec + fixtures) and is named in the
      record's `## Rule` prose — NOT a frontmatter field.
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

## 4. Gate (LAST group — run it GREEN; it reads the CHANGE)

- [ ] Run `npm run gate` after EACH group above (continuously), and as the final group. It reads
      the CHANGE — the functional-spec delta, the `standards/` record, the test markers, the full
      suite, and git — all of which exist pre-archive. It aggregates, failing on first red:
      1. project mechanical checks (full eslint + vitest suite — regression guard; the
         `eslint-plugin-local/require-rate-limit` rule runs here in `eslint .`);
      2. org-standards integrity (none pinned — skipped);
      3. standard validation — each touched/created `standards/` record has `governs:` + `tier:`
         and a `### Requirement:` with SHALL/MUST + a `conforms` scenario + a `violates` scenario;
      4. coverage — per touched functional spec AND per touched/created standard record, EVERY
         `#### Scenario:` must have ≥1 test carrying its `@spec <capability-or-standard>/<requirement>/<scenario>`
         marker (there is no `check:` to resolve; the enforcer is named in prose). A declared
         scenario with no projecting test ⇒ declared-but-unprojected ⇒ FAIL. Here that means all 4
         scenarios (2 limiter + conforms/violates) must each be marked. The gate also VERIFIES the
         standard's declared `tier:` matches what its check-test resolves to (a machine-checkable
         artifact ⇒ `mechanical`) and fails on mismatch;
      5. regression diff vs merge-base — the functional spec's built requirement+scenario set
         against its baseline, and the `standards/` records diffed in git. Both are NEW here → no
         baseline; coverage governs. NO archived main spec is needed.
      6. `openspec validate --strict` (the functional spec delta only — standards are spec-first
         artifacts, not OpenSpec specs);
      7. judge (tier-judge clauses) — none here; the judge is a local in-loop step, not a CI check.
- [ ] Confirm the gate is GREEN. (Before this group it is RED — see README: the new standard's
      check-test isn't wired yet AND the AST rule, once landed, flags the 3 existing unlimited
      handlers.)

---

**Then archive + merge (the post-green finish — not a task group, nothing runs after it).**
`openspec archive add-rate-limit` merges the `rate-limiter` delta into
`openspec/specs/rate-limiter/spec.md` (a pure native spec — requirements survive natively;
OpenSpec's own native step fills the stubbed `## Purpose`). The `standards/rate-limit-policy.md`
record is a plain git file — committed with the change, untouched by archive. Merge is the finish.
