# project.md — spec-first project contract (Node/TS: eslint + vitest)

> **Principle #0 — the spec/standard is the source of truth.** This file wires the EXTERNAL
> checks that keep every derivative (code, test, check, migration) consistent with its source.
> Durable spec-first content lives OUTSIDE the change folder and OUTSIDE openspec/specs/ — the
> functional spec archives natively, and standards live under `standards/` — so `openspec archive`
> never strips anything.

## Gate

`npm run gate` — MUST exit 0 before a change is complete, and is run after every task group
during the change (not only at the end). It reads the CHANGE (the functional-spec delta + the
`standards/` records + test markers + git) and runs PRE-archive. Runs the steps in order, failing
on the first red; the judge runs last and is non-blocking in CI.

`package.json`:

```json
{
  "scripts": {
    "gate":       "npm run check:suite && npm run check:standards && npm run check:coverage && npm run check:regression && openspec validate --strict && npm run judge:local",
    "check:suite":      "eslint . && tsc --noEmit && vitest run",
    "check:standards":  "tsx scripts/standards.ts",
    "check:coverage":   "tsx scripts/coverage.ts",
    "check:regression": "tsx scripts/regression.ts",
    "judge:local":      "tsx scripts/judge.ts --mode=local"
  }
}
```

1. **Project mechanical checks** — `eslint . && tsc --noEmit && vitest run`. The FULL existing
   suite (regression guard: greening your own checks while reddening an existing one fails). The
   `require-rate-limit` AST rule runs here, in `eslint .`, like any standard's check.

2. **Org-standards integrity** — none (no org bundle consumed). Skipped. (If an `org-standards/`
   bundle were pinned, a `scripts/org-integrity.ts` would assert each copied-in standard matches
   the pinned bundle — pin + integrity check.)

3. **Standard validation** — `tsx scripts/standards.ts`. Standards are spec-first artifacts, NOT
   OpenSpec specs, so `openspec validate` does not touch them — the gate validates each
   touched/created `standards/<name>.md` record instead: `governs:` + `tier:` present in
   frontmatter, and under `## Rule` a `### Requirement:` with a literal SHALL/MUST plus a
   `#### Scenario: conforms` and a `#### Scenario: violates`. (Frontmatter is safe here — the file
   is never archived.)

4. **Coverage** — `tsx scripts/coverage.ts`. Reads the CHANGE: the touched functional-spec deltas,
   the touched/created `standards/` records, the test markers, and git. For every touched
   functional spec AND every touched/created standard, each `#### Scenario:` MUST have ≥1 test
   carrying its `@spec <capability-or-standard>/<requirement>/<scenario>` marker; a scenario with
   no projecting test is declared-but-unprojected and FAILS (the faithful operationalization of "a
   test is a projection of a scenario"). The enforcer for a standard@mechanical is named in the
   record's `## Rule` prose, not a field — coverage keys on the markers. Branches on `tier:`:
   mechanical → grep the test tree for the marker and assert it runs; judge → assert the scenario
   maps to a registered judge run. The declared `tier:` is GATE-VERIFIED: coverage asserts it
   matches what the projecting test resolves to (a lint/AST/test artifact ⇒ mechanical; a
   registered judge run ⇒ judge) and FAILS on mismatch. Coverage is CHANGE-SCOPED — untouched
   legacy specs are not audited; touch one and you must spec + test it. (`tsx scripts/coverage.ts
   --all` is a non-blocking backlog report over the whole tree — the brownfield ratchet.)

5. **Regression diff** — `tsx scripts/regression.ts`. Diffs against the git merge-base, NO
   archived main spec required. For each touched functional spec, diff its requirement+scenario
   set against its BASELINE = the built spec at this change's merge-base
   (`openspec/specs/<cap>/spec.md` recovered from git). For each touched standard, diff its
   `standards/<name>.md` record in git. A requirement/scenario gone without an explicit
   `## REMOVED Requirements` delta (functional) or a Rationale-logged amendment (standard) fails. A
   newly added spec/standard has no baseline → coverage governs it instead.

6. **`openspec validate --strict`** — exit 0, the functional spec delta only (standards are not
   OpenSpec specs). On 1.4.1, `## Purpose` < 50 chars fails only under `--strict`; structural rules
   (Purpose + Requirements, each requirement a literal SHALL/MUST + ≥1 `#### Scenario:`) are hard
   errors regardless.

7. **Judge (tier: judge clauses)** — `tsx scripts/judge.ts`. Locally (`--mode=local`): BLOCKING —
   do not close a task with an `open` suspect. In CI (`--mode=print`): SOFT — post suspects as PR
   annotations, never auto-fail. (This change introduces no judge-tier clause, but the step is
   always wired.)

## Traceability marker

Tests tie back to specs/standards with `// @spec <capability-or-standard>/<requirement>/<scenario>`
comments, placed on each test CASE (each `it()`/`test` block, or each RuleTester `valid`|`invalid`
fixture group), one marker per scenario projected — a conforms-AND-violates check-test carries
TWO. `scripts/coverage.ts` greps these. Matching is CASE-SENSITIVE, normalizing trailing
whitespace only — keep the marker's casing exact, identical between the test and the source's
`#### Scenario:` line.

## Org-standards pin

- **Bundle:** none
- **Version:** n/a

(No org bundle consumed; the integrity gate step is skipped. Org-shared standards travel as a
copy-in `standards/` bundle — pin a version and add the integrity check when you consume one.)

## Branch protection (set at the git host — the gate canNOT enforce this)

On the default branch (`main`), require:
- no direct pushes;
- a pull request to merge;
- the gate's CI check (`gate` job in `ci.yml`) green — a **required** check;
- ≥ 1 human approval — which RATIFIES the spec/standard as the source of truth (intent + scenario
  strength for `rate-limiter`, and the `rate-limit-policy` standard CREATE), since the gate proves
  consistency among projections but not the source's correctness.

## Adoption mode

- **Mode:** greenfield
- **Strict scope:** whole repo (no legacy tree to scope around).
- **Legacy:** none. Brownfield is handled by coverage being CHANGE-SCOPED — untouched legacy specs
  are not audited; touch one and you spec + test it — backed by a non-blocking `coverage --all`
  backlog report (the ratchet). The 3 pre-existing public handlers are unspecced *code*, not
  specced-legacy; the standard + codemod bring them under spec on this change.
