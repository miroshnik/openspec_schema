# Check: `require-rate-limit` (a lint/AST projection)

Enforces the `rate-limit-policy` standard (the living record at
`standards/rate-limit-policy.md`). The standard names its enforcer in its `## Rule` prose, NOT a
frontmatter field; THIS rule is that enforcer, and the check-test below is the standard's check.
The standard's `conforms`/`violates` scenarios are these fixtures.

A lint/AST rule is ONE projection FORM, chosen here because THIS scenario is a structural AST fact
(export ⊂ public router ⇒ wrapped) — static analysis is the stack's natural check for it. It is
NOT the canonical or default mechanism: a behavioral scenario (e.g. the limiter's
`429 + Retry-After`) projects to an EXECUTABLE test instead, and other scenarios project to
unit/e2e/property/contract/type-level/runtime-assertion checks — whatever that stack uses to check
THAT scenario. `tier: mechanical` means machine-checkable-at-all, not "lint."

## Rule

`eslint-plugin-local/require-rate-limit` — an AST rule scoped to `src/routes/public/**`.

- **Invariant enforced:** every export reachable from the public router is wrapped by
  `withRateLimit`.
- **AST shape it asserts:** for each `export` (named or default) in a governed file, the
  exported expression is a `CallExpression` whose callee is the identifier `withRateLimit`
  (imported from `src/middleware/withRateLimit.ts`). A bare handler export, or an export wrapped
  by anything else, is a violation.
- **Message on violation:** registered as `meta.messages.unwrapped` (the eslint norm), e.g.
  `"export '{{name}}' is not wrapped by withRateLimit — public handlers must be rate-limited (see
  standards/rate-limit-policy.md)."` — the check-test below asserts it via `messageId: "unwrapped"`.

## Check-test — `test/standards/rate-limit-policy.test.ts`

```ts
import { RuleTester } from "eslint";
import rule from "../../src/lib/eslint-rules/require-rate-limit";

new RuleTester().run("require-rate-limit", rule, {
  // @spec rate-limit-policy/Public handlers must be rate-limited/conforms
  // valid group projects the `conforms` scenario
  valid: [
    { code: `export const getUser = withRateLimit(async (req, res) => handle(req, res));` },
  ],
  // @spec rate-limit-policy/Public handlers must be rate-limited/violates
  // invalid group projects the `violates` scenario; assert the rule FLAGS it
  invalid: [
    {
      code: `export const getUser = async (req, res) => handle(req, res);`,
      errors: [{ messageId: "unwrapped" }],
    },
  ],
});
```

One check-test, TWO `@spec` markers — one per scenario it projects, on each RuleTester fixture
group. Coverage verifies BOTH `conforms` and `violates` are projected; a scenario with no marked
group would be declared-but-unprojected and FAIL.

## Fixtures (mirror the standard's scenarios)

- `fixtures/conforms.ts` — every export wrapped by `withRateLimit` → rule passes.
- `fixtures/violates.ts` — a bare exported handler → rule flags it.

The coverage gate resolves each scenario by grepping its `@spec` marker above: every
`#### Scenario:` under the standard's `## Rule` must have ≥1 marked group projecting it. The
casing of `Public handlers must be rate-limited` (the requirement) and of `conforms`/`violates`
(the scenarios) matches the `### Requirement:` and `#### Scenario:` lines in
`standards/rate-limit-policy.md` exactly (case-sensitive, trailing-whitespace-normalized);
renaming a scenario means re-projecting its marker.
