# The project gate — paste into `openspec/project.md`

spec-first references "the project's **gate**" abstractly so the schema stays
stack-agnostic. THIS is where you make it concrete for your stack. The gate is
EXTERNAL to OpenSpec: `openspec validate --strict` is only one of its inputs.
Everything below is a MECHANICAL check you wire into a SINGLE command that both
CI and the local authoring loop run — continuously during a change, and as the
LAST TASK GROUP on the CHANGE before archive. The gate is MECHANICAL-ONLY:
deterministic, cheap, reproducible, no LLM. CI runs THIS as the required status
check. The judge is a SEPARATE LOCAL step in the authoring loop (hard-local),
not a CI gate step — see below.

The gate reads the CHANGE, pre-archive: the functional-spec deltas, the
top-level `standards/` files, the test markers, the full suite, and git. It runs
ONCE, green, before you archive. Archive + merge are the FINISH after green.

Copy the block below into `openspec/project.md` and replace the `<…>` slots.

---

```
## Gate

`<gate command>` — MUST exit 0 before a change is archived, and is run after every
task group during the change (not only at the end). It runs steps 1-5 in order and
fails on the first red. These five steps are the MECHANICAL gate — deterministic,
no LLM — run in CI as the required status check AND locally after each task group.
The judge is NOT one of these steps; it is a separate LOCAL step in the authoring
loop (hard-local), described after the gate. The gate runs on the CHANGE as the
LAST TASK GROUP — everything it reads (deltas + `standards/` + tests + git) exists
pre-archive.

1. Project mechanical checks — `<lint/type/test command>`. The functional-regression guard —
   the FULL existing suite, not only this change's checks. (Greening your own checks while
   reddening an existing one broke existing functionality — a regression — fail.)

2. Org-standards integrity — `<integrity command>` (only if you consume an org bundle;
   skip otherwise). Every file under `standards/_org/` MUST match the pinned org-standards
   bundle on its NORMALIZED rule-bearing fields (frontmatter governs/tier + the
   requirement/scenario set, ignoring whitespace/comments) — raw byte-match flips on
   reformatting. `standards/_org/` is a copied-in bundle, never authored locally, so nothing
   rewrites it. A locally loosened org standard is a forbidden divergence — fail. (To change
   an org rule, upstream a REVISE and bump the pin; you cannot weaken it locally.) The portable
   unit is the STANDARD'S RULE — its `### Requirement:` + conforms/violates SCENARIOS — which is
   stack-agnostic; org-integrity matches that. Each consuming project PROJECTS those scenarios
   into its OWN enforcement form (its own LOCAL test) — you share the SCENARIO, not a particular
   lint rule, so the projection/test is not what integrity matches.

3. Coverage — `<coverage command>`. Reads the CHANGE: the functional-spec deltas, the
   `standards/` files touched/created by this change, the test markers, and git. PRE-archive.
   For every functional spec TOUCHED BY THIS CHANGE (added/modified delta scenarios) AND every
   `standards/<name>.md` file TOUCHED OR CREATED by this change, EVERY `#### Scenario:` MUST
   have ≥1 test carrying its `@spec <capability-or-standard>/<requirement>/<scenario>` marker —
   a scenario with no projecting test is declared-but-unprojected, fail. (This is the faithful
   operationalization of "a test is a PROJECTION of a scenario.") Coverage keys on the SCENARIO
   markers; a standard@mechanical names its enforcer (whatever its stack uses for THAT scenario)
   in the test/standard body and prose, not in frontmatter. Untouched functional specs and
   untouched `standards/` files are not re-audited here. A judge-tier clause is covered by its
   registered judge check, not a mechanical test. (Implement once, branching on a standard's
   `tier:` against what its test resolves to, NOT on a check field: mechanical → grep the test
   tree for the per-scenario markers, assert each scenario's test exists and runs; judge → assert
   the scenario names a registered, re-runnable judge check. A separate `<coverage --all>` is a
   non-blocking backlog report for the brownfield ratchet.) HONESTY: coverage proves each SCENARIO
   is LINKED to a wired-in test that runs — it does NOT prove the test faithfully/non-trivially
   exercises the scenario (an empty test with the right marker passes). Test STRENGTH is
   ratification's job.
   - tier is GATE-VERIFIED (for standards): assert the declared `tier:` means
     machine-checkable-at-all (mechanical) vs needs-human-judgment (judge) — NOT a mechanism —
     and matches what the scenario's test actually resolves to: a mechanical artifact (an
     executable unit/e2e/property/contract test, a type-level check, a runtime assertion, OR a
     lint/AST rule — whatever that stack uses) ⇒ `mechanical`, a registered judge run ⇒ `judge`.
     FAIL on mismatch (the same discipline that derives binding, not a thing the author can assert
     past the gate).
   - Standards validity (the gate validates `standards/`, not `openspec validate`): each
     touched/created `standards/<name>.md` MUST carry frontmatter `governs:` + `tier:`, and a
     `### Requirement:` with literal SHALL/MUST plus a `#### Scenario: conforms` and a
     `#### Scenario: violates`. Standards are spec-first artifacts outside `openspec/`; strict
     never sees them, so the gate is their structural check — fail if malformed.

4. No silent scenario drop (spec/standard preservation) — `<preservation command>`. Diff against
   the git MERGE-BASE. For each touched functional spec, diff its requirement+scenario set against
   its BASELINE = the full built spec at this change's merge-base (`openspec/specs/<cap>/spec.md`
   before these deltas applied — from git). For each touched `standards/<name>.md`, diff its
   requirement+scenario set against the same file at the merge-base (a plain git diff — standards
   are ordinary git files). A requirement or scenario that disappeared without an explicit
   `## REMOVED Requirements` (or `## RENAMED Requirements`) delta — or, for a standard, without a
   Rationale-log entry recording the removal — is a silent drop, fail. (The full suite cannot catch
   this: a scenario deleted together with its test leaves the suite green.) A touched spec or a
   newly created standard with NO form at the merge-base has no baseline — coverage governs it
   instead, and the preservation baseline is established from this change forward.

5. `openspec validate --strict` — exit 0, on the FUNCTIONAL specs (capabilities). Tested against
   OpenSpec 1.4.1: `## Purpose` < 50 chars fails ONLY under `--strict`; the structural rules (a
   spec needs `## Purpose` + `## Requirements`, each requirement a literal SHALL/MUST + ≥1
   `#### Scenario:`) are HARD errors that fail even without `--strict`. (`## Why` < 50 chars and
   > 10 deltas do NOT fail strict in 1.4.1 — if you want those bars, add them as your own
   grep/length sub-step, not as a `--strict` claim.) Standards live outside `openspec/specs/` and
   are NOT validated here — step 3 validates them. On adoption, scope strict to changed paths until
   the repo is brought strict-clean once (SETUP Step 0), so an unrelated change doesn't red on
   legacy well-formedness.

The five steps above are the whole mechanical gate. The judge below is NOT a gate step and does
NOT run in CI.

## The judge — a LOCAL in-loop reviewer (hard-local), not a CI step

Judge (standards with `tier: judge`) — `<judge command>`: an agent in print mode reviewing the
governed scope for the part no mechanical check covers. It SURFACES suspects + standard-candidates
for the un-mechanizable part; it gives no verdict — a human decides.
- It runs in the AUTHORING LOOP, locally (hard-local): BLOCKING there. Do not close a task while
  the judge has an `open` suspect — fix it, or mark it `justified` with a pointer to the standard's
  Rationale log.
- It is NOT a CI job and NOT a gate step that runs in CI. CI stays mechanical-only (deterministic,
  no API keys). A team MAY add their own NON-required judge-annotation job that posts LLM suspects
  on PRs, but it is not shipped by default (see templates/judge.md and the example ci.yml).
- RATIFICATION is the human reviewer at PR review: they read the diff and judge the un-mechanizable
  part (intent, naming, ergonomics, spirit) themselves, and MAY run the judge on demand. That is
  where trust in the source is established.

## Traceability marker

Tests tie back to functional specs AND standards with `<marker convention>` (default: a
`@spec <capability-or-standard>/<requirement>/<scenario>` comment in the test file). The SCENARIO
is the traceable/generative unit; the marker sits on each test CASE (each it()/test block, or each
RuleTester valid|invalid fixture group), and a single test MAY carry MULTIPLE `@spec` markers —
one per scenario it projects (a standard@mechanical check-test covering conforms AND violates
carries TWO). The coverage check greps these. Scenario names are matched like requirement names:
case-sensitive, trailing-whitespace normalized — keep the marker's casing exact, renaming a
scenario means re-projecting its marker, and bind the preservation matcher (the merge-base diff)
to the same trailing-only, case-sensitive normalization (in 1.4.1 archive matches requirement
names that way too).
```

---

## Archive + merge are the finish

Functional-spec deltas archive NATIVELY into `openspec/specs/` (requirements + scenarios survive
— they are plain OpenSpec). A brand-new spec gets OpenSpec's own stubbed `## Purpose` ("TBD …
Update Purpose after archive"); filling it is OpenSpec's OWN native step. Standards live in
top-level `standards/` as ordinary git files and are never touched by archive. So the gate runs
ONCE on the CHANGE, green, as the last task group; archive merges the functional-spec deltas into
`openspec/specs/` and moves the change to `changes/archive/`. Merge is the finish.

## Governance standards run here too

If you adopt `templates/governance.md` (all-changes-via-flow, branch-per-change, lifecycle — as
`standards/` records), their enforcers run in step 1 like any standard — no extra gate step.
The one thing the gate can NOT do is block direct pushes to the default branch; that is your git
host's branch protection. Wire both.

## Why this lives in `project.md`, not the schema

The schema's `instruction:` guidance has the agent AUTHOR each clause, its enforcer, its tier,
and its test. WHAT command runs them is stack-specific, so it belongs to the consuming project.
Keeping the gate here is what lets one copy of the spec-first flow serve every project: the
universal workflow stays in the templates; the project-specific contract (gate command + org-pin
+ branch-protection + adoption mode) lives in `project.md`.
