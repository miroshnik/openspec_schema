# The project gate — paste into `openspec/project.md`

spec-first references "the project's **gate**" abstractly so the schema stays
stack-agnostic. THIS is where you make it concrete for your stack. The gate is
EXTERNAL to OpenSpec: `openspec validate --strict` is only one of its inputs.
Everything below is a mechanical check you wire into a SINGLE command that both
CI and the local authoring loop run — continuously during a change, and as the
last task group before the change is complete.

Copy the block below into `openspec/project.md` and replace the `<…>` slots.

---

```
## Gate

`<gate command>` — MUST exit 0 before a change is complete, and is run after every
task group during the change (not only at the end). It runs steps 1-5 in order and
fails on the first red; step 6 (the judge) runs last and is non-blocking in CI (see
step 6). Locally the loop treats an open judge suspect as not-yet-green too.

1. Project mechanical checks — `<lint/type/test command>`. The FULL existing suite,
   not only this change's checks. (Regression guard: greening your own checks while
   reddening an existing one is a regression — fail.)

2. Org-standards integrity — `<integrity command>` (only if you consume an org bundle;
   skip otherwise). Every spec under `specs/_org/` MUST match the pinned org-standards bundle
   on its NORMALIZED rule-bearing fields (frontmatter governs/test/tier + the
   requirement/scenario set, ignoring whitespace/comments) — raw byte-match flips on reformatting.
   `specs/_org/` is never a local delta target, so archive never rewrites it. A locally loosened
   org standard is a forbidden divergence — fail. (To change an org rule, upstream a REVISE and
   bump the pin; you cannot weaken it locally.) The portable unit is the STANDARD'S SPEC — its
   rule + conforms/violates SCENARIOS — which is stack-agnostic and immutable; org-integrity
   checks that shared spec under `specs/_org/`. Each consuming project PROJECTS those scenarios
   into its OWN enforcement form (its own LOCAL test) — you share the SCENARIO, not a particular
   lint rule, so the projection/test is not what integrity matches.

3. Coverage — `<coverage command>`. For every spec TOUCHED BY THIS CHANGE (added/modified
   delta), EVERY `#### Scenario:` MUST have ≥1 test carrying its
   `@spec <capability>/<requirement>/<scenario>` marker — a scenario with no projecting test is
   declared-but-unprojected, fail. (This is stricter than the old requirement-level linkage and
   is the faithful operationalization of "a test is a PROJECTION of a scenario.") Coverage keys
   on the SCENARIO markers resolved from `test:` ONLY (there is no `check:` to resolve — a
   standard@mechanical names its enforcer, whatever its stack uses for THAT scenario, in the
   test/check body and prose, not in frontmatter). Untouched specs are not re-audited here; a
   spec marked `adoption: legacy` is exempt until first modified (then the exemption drops). A
   judge-tier clause is covered by its registered judge check, not a mechanical test. (Implement
   once, branching on `tier:` against what `test:` is, NOT on a check: field: mechanical → grep
   the test tree for the spec's per-scenario markers, assert each scenario's test exists and
   runs; judge → assert `test:` names a registered, re-runnable judge check. A separate
   `<coverage --all>` is a non-blocking backlog report.) HONESTY: coverage proves each SCENARIO
   is LINKED to a wired-in test that runs — it does NOT prove the test faithfully/non-trivially
   exercises the scenario (an empty test with the right marker passes). Test STRENGTH is
   ratification's job.
   - tier is GATE-VERIFIED: assert the declared `tier:` means machine-checkable-at-all
     (mechanical) vs needs-human-judgment (judge) — NOT a mechanism — and matches what `test:`
     actually resolves to: a mechanical artifact (an executable unit/e2e/property/contract test,
     a type-level check, a runtime assertion, OR a lint/AST rule — whatever that stack uses) ⇒
     `mechanical`, a registered judge run ⇒ `judge`. FAIL on mismatch (the same discipline that
     derives binding, not a thing the author can assert past the gate).
   - RE-APPLY sub-check: after a NEW spec's first (CREATE) archive, the built
     `openspec/specs/<name>/spec.md` MUST still carry its CORE (`test:`). CREATE rebuilds the
     spec from the ADDED delta alone and DROPS the frontmatter CORE (1.4.1), so a forgotten
     re-apply leaves a just-archived NEW spec with no `test:` — FAIL, so it cannot silently
     pass coverage. (The human-facing re-apply note in tasks still stands; this backstops it.)

4. Regression diff — `<regression command>`. For each touched main spec, diff its
   requirement+scenario set against its BASELINE = the full built spec at this change's
   merge-base (`openspec/specs/<cap>/spec.md` before these deltas applied — from git, NOT
   the archive delta, which is only the diff and cannot reconstruct the prior set). A
   requirement or scenario that disappeared without an explicit `## REMOVED Requirements`
   (or `## RENAMED Requirements`) delta is a silent regression (archive's MODIFIED replaces
   the whole requirement block) — fail. A touched spec with NO built form at the merge-base
   (newly added, or the FIRST modify of an `adoption: legacy` spec) has no baseline — coverage
   governs it instead, and the regression baseline is established from this change forward.

5. `openspec validate --strict` — exit 0. Tested against OpenSpec 1.4.1: `## Purpose` < 50 chars
   fails ONLY under `--strict`; the structural rules (a spec needs `## Purpose` + `## Requirements`,
   each requirement a literal SHALL/MUST + ≥1 `#### Scenario:`) are HARD errors that fail even
   without `--strict`. (`## Why` < 50 chars and > 10 deltas do NOT fail strict in 1.4.1 — if you
   want those bars, add them as your own grep/length sub-step, not as a `--strict` claim.) On
   adoption, scope strict to changed paths until the repo is brought strict-clean once (SETUP
   Step 0), so an unrelated change doesn't red on legacy well-formedness.

6. Judge (tier: judge clauses) — `<judge command>`: an agent in print mode reviewing the
   governed scope for the part no mechanical check covers.
   - Locally (authoring loop): BLOCKING. Do not close a task while the judge has an `open`
     suspect — fix it, or mark it `justified` with a pointer to the spec's Rationale log.
   - In CI: SOFT. Post suspects as PR annotations or a "human-review-needed" flag. NEVER
     auto-fail the build on a non-deterministic verdict (see templates/judge.md).

## Traceability marker

Tests tie back to specs with `<marker convention>` (default: a
`@spec <capability>/<requirement>/<scenario>` comment in the test file). The SCENARIO is the
traceable/generative unit; the marker sits on each test CASE (each it()/test block, or each
RuleTester valid|invalid fixture group), and a single test MAY carry MULTIPLE `@spec` markers —
one per scenario it projects (a standard@mechanical check-test covering conforms AND violates
carries TWO). The coverage check greps these. Scenario names are matched like requirement names:
case-sensitive, trailing-whitespace normalized — keep the marker's casing exact, renaming a
scenario means re-projecting its marker, and bind the regression matcher to the same
trailing-only, case-sensitive normalization (in 1.4.1 archive matches requirement names that
way too).
```

---

## Governance standards run here too

If you adopt `templates/governance.md` (all-changes-via-flow, branch-per-change, lifecycle),
their enforcers run in step 1 like any standard — no extra gate step. The one thing the
gate can NOT do is block direct pushes to the default branch; that is your git host's branch
protection. Wire both.

## Why this lives in `project.md`, not the schema

The schema's `instruction:` guidance has the agent AUTHOR each clause, its enforcer, its tier,
and its test. WHAT command runs them is stack-specific, so it belongs to the consuming project.
Keeping the gate here is what lets one copy of the spec-first flow serve every project: the
universal workflow stays in the templates; the project-specific contract (gate command + org-pin
+ branch-protection + adoption mode) lives in `project.md`.
