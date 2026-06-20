# project.md — spec-first project contract (Node/TS: eslint + vitest)

> **Principle #0 — the spec is the source of truth.** This file wires the EXTERNAL checks that
> keep every derivative (code, test, check, migration) consistent with its spec.

## Gate

`npm run gate` — MUST exit 0 before a change is complete, and is run after every task group
during the change (not only at the end). Runs steps 1-5 in order, failing on the first red;
step 6 (the judge) runs last and is non-blocking in CI.

`package.json`:

```json
{
  "scripts": {
    "gate":       "npm run check:suite && npm run check:coverage && npm run check:regression && openspec validate --strict && npm run judge:local",
    "check:suite":      "eslint . && tsc --noEmit && vitest run",
    "check:coverage":   "tsx scripts/coverage.ts",
    "check:regression": "tsx scripts/regression.ts",
    "judge:local":      "tsx scripts/judge.ts --mode=local"
  }
}
```

1. **Project mechanical checks** — `eslint . && tsc --noEmit && vitest run`. The FULL existing
   suite (regression guard: greening your own checks while reddening an existing one fails). The
   `require-rate-limit` AST rule runs here, in `eslint .`, like any standard's check.

2. **Org-standards integrity** — none (no org bundle consumed). Skipped. (If `specs/_org/`
   existed, a `scripts/org-integrity.ts` would assert each matches the pinned bundle.)

3. **Coverage** — `tsx scripts/coverage.ts`. For every spec TOUCHED BY THIS CHANGE, its `test:`
   MUST resolve to a real, wired-in test found via its `@spec` marker (the CORE keys on `test:`
   only — there is no `check:` to resolve). Per touched spec, every `#### Scenario:` MUST have ≥1
   test carrying its `@spec <capability>/<requirement>/<scenario>` marker; a scenario with no
   projecting test is declared-but-unprojected and FAILS (the faithful operationalization of "a
   test is a projection of a scenario", stricter than the old requirement-level linkage). Branches
   on `tier:`: mechanical → grep the test tree
   for the marker and assert it runs; judge → assert `test:` names a registered judge run. The
   declared `tier:` is GATE-VERIFIED: coverage asserts it matches what `test:` resolves to (a
   lint/AST/test artifact ⇒ mechanical; a registered judge run ⇒ judge) and FAILS on mismatch.
   This step also backstops the post-archive re-apply: after a NEW spec's CREATE archive, a built
   spec missing its CORE (`test:`) FAILS coverage, so a forgotten re-apply cannot silently pass.
   (`tsx scripts/coverage.ts --all` is a non-blocking backlog report over the whole tree.)

4. **Regression diff** — `tsx scripts/regression.ts`. For each touched main spec, diff its
   requirement+scenario set against its BASELINE = the full built spec at this change's
   merge-base (`openspec/specs/<cap>/spec.md` recovered from git — NOT the archive delta). A
   requirement/scenario gone without an explicit `## REMOVED Requirements` delta fails. A newly
   added spec has no baseline → coverage governs it instead.

5. **`openspec validate --strict`** — exit 0. On 1.4.1, `## Purpose` < 50 chars fails only under
   `--strict`; structural rules (Purpose + Requirements, each requirement a literal SHALL/MUST +
   ≥1 `#### Scenario:`) are hard errors regardless.

6. **Judge (tier: judge clauses)** — `tsx scripts/judge.ts`. Locally (`--mode=local`): BLOCKING —
   do not close a task with an `open` suspect. In CI (`--mode=print`): SOFT — post suspects as PR
   annotations, never auto-fail. (This change introduces no judge-tier clause, but the step is
   always wired.)

## Traceability marker

Tests tie back to specs with `// @spec <capability>/<requirement>/<scenario>` comments, placed on
each test CASE (each `it()`/`test` block, or each RuleTester `valid`|`invalid` fixture group), one
marker per scenario projected — a conforms-AND-violates check-test carries TWO. `scripts/coverage.ts`
greps these. Matching is CASE-SENSITIVE, normalizing trailing whitespace only — keep the marker's
casing exact, identical between the test and the spec's `#### Scenario:` line.

## Org-standards pin

- **Bundle:** none
- **Version:** n/a

(No org bundle consumed; the integrity gate step is skipped.)

## Branch protection (set at the git host — the gate canNOT enforce this)

On the default branch (`main`), require:
- no direct pushes;
- a pull request to merge;
- the gate's CI check (`gate` job in `ci.yml`) green — a **required** check;
- ≥ 1 human approval — which RATIFIES the spec as the source of truth (intent + scenario
  strength for `rate-limiter`, and the `rate-limit-policy` standard CREATE), since the gate
  proves consistency among projections but not the source's correctness.

## Adoption mode

- **Mode:** greenfield
- **Strict scope:** whole repo (no legacy tree to scope around).
- **Legacy:** none — every spec is active (no spec carries `adoption: legacy`; the field is
  OMITTED on active specs and present only to exempt a pre-existing spec from coverage until
  first modified). The 3 pre-existing public handlers are unspecced *code*, not specced-legacy;
  the standard + codemod bring them under spec on this change.
