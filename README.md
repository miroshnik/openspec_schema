# spec-first — an OpenSpec schema

**Principle #0 — the spec is the single source of truth.** Code, tests, checks, migrations,
and docs are all *projections* of the durable record: a test projects a scenario — the scenario
(Given/When/Then, or conforms/violates) is the canonical, stack-agnostic statement of the
property, and the test is its local, regenerable artifact (when the scenario changes, RE-PROJECT
its test). The projection's FORM is stack-specific — a unit / e2e / property / contract /
type-level / runtime-assertion test, or a lint/AST rule — whatever that stack uses to check THAT
scenario. A migration projects the change; code is born in `apply` from tasks (which project from
the record). On any divergence the record is authoritative and changes FIRST — you never alter a
derivative behind its spec's back. Everything else in this README — the two shapes (functional
spec, standard), the flow order, the per-scenario test, the gate, the all-via-flow governance —
exists to enforce that one rule.

> **An OpenSpec flow with checks you can trust your generation against, plus standards that
> are mandatory across the whole project — enforced, not filed in a wiki.**

---

## Table of contents

1. [The problem](#the-problem)
2. [What the gate does and doesn't prove (honesty)](#what-the-gate-does-and-doesnt-prove)
3. [The model — two shapes](#the-model--two-shapes)
4. [The flow & lifecycle](#the-flow--lifecycle)
5. [The standards loop](#the-standards-loop)
6. [The gate](#the-gate)
7. [Tests, coverage, regression](#tests-coverage-regression)
8. [Empirical OpenSpec 1.4.1 facts](#empirical-openspec-141-facts)
9. [Governance — make the flow the only path](#governance--make-the-flow-the-only-path)
10. [Brownfield adoption](#brownfield-adoption)
11. [Org-shared standards](#org-shared-standards)
12. [Worked end-to-end example](#worked-end-to-end-example)
13. [What's inside](#whats-inside)
14. [Install](#install)
15. [The project.md contract](#the-projectmd-contract)
16. [Cross-project / team use](#cross-project--team-use)
17. [Versioning & dogfooding](#versioning--dogfooding)

---

## The problem

AI agents generate code fast — but you can only trust what you can **check**, and the rules
that actually define a codebase (how it does auth, errors, logging, layering) live in people's
heads or a wiki nobody reads. **spec-first** makes both first-class:

- **Functional behavior** lives as pure OpenSpec specs under `openspec/specs/`; **cross-cutting
  standards** live as self-contained living records under top-level `standards/`.
- **Every scenario projects to a test** — each `#### Scenario:` becomes a stack-specific test whose
  FORM is the stack's choice (unit / e2e / property / contract / type-level / runtime-assertion
  test, or a lint/AST rule) — a behavioral scenario typically to an executable test, a purely
  structural rule MAY project to a lint/AST check.
- A single **gate** must be green before a change completes (full suite + org-integrity +
  coverage + regression + `validate --strict`). The gate CI runs is mechanical-only; the judge
  is a separate local step.
- What no machine can check, an **LLM judge** surfaces — a LOCAL in-loop reviewer (hard-local);
  it is not a CI job. CI is mechanical-only; the human ratifies at review.
- A **standard** is a cross-cutting rule that binds every surface it governs across the *whole
  project* — enforced by the gate, not filed in a wiki. (It can optionally be shared across
  projects too — a bonus, below — but that is not the point.)

Adoption is incremental: spec-first governs **changes**, not your existing tree — the gate
ratchets coverage upward one change at a time and stays green throughout. It is a **community
schema** — an overlay you fork into your project (built on OpenSpec's `spec-driven`), so it
inherits the delta-spec format that `archive` / `sync` rely on.

## What the gate does and doesn't prove

This is the honest line, and it is the whole point of the design:

**Green means** the spec, its standards, the tests, and the code are CONSISTENT, and that no
scenario silently regressed. It makes the spec the thing you *review*. It does NOT prove the spec
is correct or its test strong: the same agent may author spec, standard, scenario, and test, so a
hollow self-consistent spec also goes green. The spec becomes trustworthy only when a human
**ratifies** it at PR review. The gate makes divergence-from-the-record mechanical;
**ratification** makes the record itself authoritative.

So the trust split is:

| What | Who/what guarantees it |
| --- | --- |
| Projections are consistent with the record | the gate (mechanical) |
| No scenario silently disappeared | the gate (regression diff) |
| Every scenario is actually projected | the gate (coverage) |
| The un-mechanizable part fits the spirit | the judge (surfaces; human ratifies) |
| **The spec/standard itself is true / its tests are strong** | **a human, at PR approval** |

Coverage proves each SCENARIO is LINKED to a marked, wired-in test that RUNS — it cannot prove the
test faithfully or non-trivially exercises the scenario (an empty `it()` with the right `@spec`
marker passes). It is linkage, not strength. Test *strength*, like spec *truth*, is what
ratification at PR review checks.

---

## The model — two shapes

spec-first has **two shapes of durable record**, and the design point is *where they live*:
durable spec-first content lives OUTSIDE the change folder, so `openspec archive` never strips it.

- **Functional capability** — a **PURE native OpenSpec spec** at
  `openspec/specs/<cap>/spec.md`. A `## Purpose` section + a `## Requirements` section
  (`### Requirement:` with literal SHALL/MUST + `#### Scenario:` Given/When/Then). It is an
  ordinary OpenSpec spec, so it survives archive *natively*. Its `## ADDED Requirements` delta
  rebuilds the main spec on archive, and its scenarios become its tests (tied via `@spec`
  markers). A brand-new spec is archived with OpenSpec's own stubbed Purpose — "TBD … Update
  Purpose after archive"; you fill that in after archive.
- **Standard** — a **self-contained living file** at top-level `standards/<name>.md` (NOT an
  OpenSpec spec, a committed git file that archive does not touch). Everything about the standard —
  scope, tier, the decision and why, the rule, its scenarios, its amendment history — lives HERE,
  always visible in one file. Shape:

  ```
  ---
  governs: <scope>            # what surfaces this rule binds
  tier:    mechanical | judge # machine-checkable vs needs-human-judgment
  ---
  # <Standard name>

  ## Decision Record          # the ADR
  Status: <proposed | accepted | superseded>
  Context: …
  Decision: …
  Consequences: …
  Alternatives considered: …

  ## Rule
  ### Requirement: <name>
  The system SHALL <normative clause — literal SHALL/MUST>.
  #### Scenario: conforms
  … an example that obeys the rule (a test fixture) …
  #### Scenario: violates
  … an example that breaks the rule (a test fixture) …

  ## Rationale log
  (<date>): <why this rule, in one or two lines>
  ```

  A standard's `governs:` scope is what makes it cross-cutting. `tier` is machine-checkable
  (mechanical) vs needs-human-judgment (judge) — it is the JUDGMENT class, NOT the mechanism. The
  `### Requirement:` SHALL/MUST clause IS the authoritative one-liner; the concrete enforcer (a
  unit/contract/type/runtime test, or a lint/AST rule id like
  `eslint-plugin-local/require-rate-limit` — the stack's choice) is named in the test body and the
  standard's prose, not in a frontmatter field. The `## Rationale log` accretes a dated line per
  revision.

**RETRIEVE reads `standards/` directly; the gate reads its scenarios for coverage.** Standards are
NOT validated by `openspec validate` — they are spec-first artifacts, not OpenSpec specs. The
**gate** validates them: `governs:` + `tier:` present; a `### Requirement:` with literal SHALL/MUST
+ a `conforms` scenario + a `violates` scenario.

### Tiers — two, no escape hatch

- **mechanical** — AST / lint / type / test. The hard-binding tier. Mechanize first.
- **judge** — focused per-standard LLM review of the part a machine cannot catch (intent /
  naming / ergonomic fit). REQUIRED to RUN and SURFACE (cannot be skipped), but it does NOT
  self-enforce: it is a **LOCAL in-loop reviewer (hard-local)** that blocks closing the task,
  and is NOT a CI job. CI is mechanical-only. The judge is BINDING only when a human ratifies its
  suspects at PR review. Every judge-tier standard MUST register a runnable judge check.

Each rule is a PAIR: the authoring guidance the agent follows (the standard's prose) + an external
mechanical check the project's **gate** runs. OpenSpec has no native
`governs:`/coverage/test-per-scenario/gate primitive, which is exactly why the teeth are external.

---

## The flow & lifecycle

A change moves through the OpenSpec artifact DAG:

```
proposal → {specs, design}; design → standard; {specs, standard} → tasks → (judge)
```

Linearized for an author: `proposal → specs → design → (standard, when a cross-cutting decision
arises) → tasks`.

- **proposal** — what changes and why; plus **Affected surfaces & rollback** (design uses the
  surfaces to retrieve the standards whose `governs:` covers them).
- **specs** — author functional capabilities as pure OpenSpec specs (deltas). No frontmatter.
- **design** — the technical design + the **standards loop** (below). design DECIDES standards.
- **standard** — author/update the self-contained `standards/<name>.md` record, generated to
  TOP-LEVEL `standards/` (OUTSIDE `openspec/`, so archive never touches it). **CONDITIONAL:** the
  standard artifact is authored/updated only when the design loop flags CREATE/REVISE; otherwise a
  no-op (no new `standards/` file). Writing the standard in design would invert #0 (a derivative,
  the recorded decision, preceding its source record).
- **tasks** — EVERYTHING in ONE pass: author the functional-spec deltas + the `standards/` records
  + tests + checks + migration. The LAST TASK GROUP runs the GATE green. **SOURCE-FIRST discipline
  (#0):** to change behavior, amend the spec/scenario (or the standard's rule) FIRST, then
  re-project its test and code — never edit a derivative behind a scenario you didn't touch.
- **judge** — optional artifact (`requires: [tasks]`), re-runnable; reviews the implemented diff.

The gate reads the CHANGE — the functional-spec deltas + the `standards/` files + the tests + the
full suite + git — and everything it needs exists PRE-archive. Archive is the FINISH: it merges
the functional-spec deltas into `openspec/specs/` and moves the change to `changes/archive/`.

The full git lifecycle (from `templates/governance.md`):

```
0. (brownfield) adoption mode set — see Brownfield adoption.
1. START   — openspec new change <id>; branch: git worktree add ../<id> -b change/<id>
2. AUTHOR  — proposal → specs → design → (standard, when a cross-cutting decision arises).
3. BUILD   — tasks in ONE pass: functional deltas + standards/ records + tests + checks +
             migration; the LAST task group runs the GATE green (it reads the change:
             deltas + standards/ + tests + full suite + git — all pre-archive). Resolve or
             justify every hard-local judge suspect.
4. ARCHIVE — openspec archive <id> → merges the functional-spec DELTAS into openspec/specs/
             (requirements survive natively) and moves the change to changes/archive/;
             standards/ files are plain committed git files, UNTOUCHED by archive.
5. PR      — open the PR from change/<id>. CI runs the mechanical gate (required check).
6. REVIEW  — a human reviewer ratifies: reads the diff, judges the un-mechanizable part, signs off
             on any standards CREATE/REVISE (and the record as source of truth); MAY run the judge on demand.
7. MERGE   — squash-merge to the default branch. Direct pushes are blocked at the host.
```

### The standards loop

Before writing the design, and whenever a cross-cutting decision arises:

- **RETRIEVE** — read top-level `standards/` directly and pull the standards whose `governs:`
  matches the surfaces this change touches (from the proposal). Apply them. Retrieval is first-try
  efficiency; the gate is the backstop for misses.

Then for each cross-cutting decision:

- **COMPLY** — fits an existing standard → follow it.
- **CREATE** — hits ungoverned ground → IDENTIFY the need for a new standard and specify its
  rule / enforcer / tier / scenarios + its Decision Record. design DECIDES; the `standard`
  artifact AUTHORS the new `standards/<name>.md`. Bias to FEW — a standard taxes every future
  change.
- **REVISE** — in tension with an existing standard → decide the DELTA (what loosens / tightens /
  carves out) and its justification; the `standard` artifact edits the file in place and appends a
  dated entry to the standard's Rationale log. Distinguish comply-vs-revise honestly; loosening
  needs strong justification.
- **DEFER** — looks cross-cutting, but bias-to-FEW says it is too early to standardize (one
  surface; not clearly a class yet). KEEP it LOCAL — it stays the functional capability's behavior
  (its scenarios). RECORD it as a standard CANDIDATE (a line in the proposal or design). PROMOTE
  it later via CREATE + a migration when a second surface needs the same rule, or a reviewer flags
  it. A standard is a rule that has OUTGROWN one capability; until then it lives as spec behavior.
  You do not standardize on a hunch.

**RESOLVE-THEN-AUTHOR.** For each clause, RECORD whether its test will be REUSED (an existing
project check/test already covers it) or newly AUTHORED. Resolve against existing project checks
first; only author when none fits.

**RATIFICATION.** Every CREATE / REVISE is surfaced for human sign-off at PR review: a mechanical
check (deterministic — `git diff` touches `standards/`) flags the standards change in the diff, and
the human reviewer ratifies it at PR review. Never self-merge a standards change — standards bind
the whole project. The local judge ALSO surfaces standard-CANDIDATES: a recurring local pattern that
should be enforced project-wide, flagged for promotion (a standard-candidate, like a
standards-changed suspect) — the DEFER-to-CREATE ratchet.

---

## The gate

spec-first references "the project's **gate**" abstractly so the schema stays stack-agnostic.
The gate is **EXTERNAL** to OpenSpec — `openspec validate --strict` is only one of its inputs.
It is a SINGLE command that both CI and the local authoring loop run: **continuously during a
change, and required green as the LAST task group before the change is complete** — PRE-archive,
when the change (deltas + `standards/` + tests + git) is all on disk.

It runs **six mechanical steps** and fails on the first red. These steps are the MECHANICAL gate —
deterministic, no LLM — run in CI as the required check AND locally after each task group. The judge
is a SEPARATE LOCAL step in the authoring loop (hard-local), not part of the CI gate (see below):

1. **Project mechanical checks** — `<lint/type/test command>`. The FULL existing suite, not only
   this change's checks. (Governance standards' enforcer commands run here too, like any standard
   — no extra step.)
2. **Org-standards integrity** — `<integrity command>` (skip if you consume no org bundle). Every
   file under `standards/_org/` MUST match the pinned org-standards bundle on its **NORMALIZED**
   rule-bearing fields (`governs`/`tier` frontmatter + the requirement/scenario set, ignoring
   whitespace/comments — raw byte-match flips on reformatting). A locally loosened org standard is
   a forbidden divergence — fail.
3. **Standards validation** — every `standards/<name>.md` is well-formed: `governs:` + `tier:`
   present; a `### Requirement:` with a literal SHALL/MUST + a `conforms` scenario + a `violates`
   scenario. (Standards are NOT seen by `openspec validate` — the gate is their validator.)
4. **Coverage** — `<coverage command>`. Reads the CHANGE: the functional-spec deltas + every
   touched/created `standards/` file + the test markers + git. Per touched functional spec AND per
   touched/created standard, EVERY `#### Scenario:` MUST have ≥1 test carrying that scenario's
   `@spec <capability-or-standard>/<requirement>/<scenario>` marker. A scenario with no projecting
   test ⇒ declared-but-unprojected ⇒ fail. The gate also asserts each standard's declared `tier`
   matches what its test resolves to (machine-checkable test ⇒ mechanical; registered judge run ⇒
   judge) and FAILS on mismatch. A judge-tier standard is covered by its registered judge check,
   not a mechanical test. Untouched functional specs are not re-audited.
5. **Regression diff** — `<regression command>`. For each touched functional spec, diff its
   requirement+scenario set against its BASELINE = the **full built spec at this change's
   merge-base** (recovered from git, NOT from the archive delta). For each touched standard, diff
   the `standards/` file against its git merge-base form. A requirement or scenario that
   disappeared without an explicit `## REMOVED Requirements` (or `## RENAMED Requirements`) delta
   — or, for a standard, a dropped requirement/scenario without a Rationale-log entry — is a silent
   regression. NO archived main spec is needed (the merge-base built form comes from git).
6. **`openspec validate --strict`** — exit 0 over the functional-spec deltas. (Structural rules
   are HARD errors even without `--strict`; `## Purpose` < 50 chars fails only under `--strict` —
   see the 1.4.1 facts below.)
**The judge — a SEPARATE LOCAL step, not part of the CI gate** (tier: judge standards) —
`<judge command>`: an agent in print mode reviewing the governed scope for the part no mechanical
check covers. It runs in the authoring loop, surfaces suspects (no verdict — a human decides), and
is NOT a CI job.
   - **Hard-local:** BLOCKING in the loop. Do not close a task while the judge has an `open`
     suspect — fix it, or mark it `justified` with a pointer to the standard's Rationale log.
   - **At PR review:** the human reviewer ratifies the un-mechanizable part themselves, and MAY run
     the judge on demand. CI itself stays mechanical-only — no LLM, no non-deterministic verdict
     reds a shared build. (A team MAY add its own non-required judge-annotation job that posts
     suspects on PRs, but that is an opt-in add-on, not shipped by default.)

**Green-as-last-group.** The gate runs throughout the change so breakage surfaces early; the
FINAL task group is "the full gate green," and it MUST be green before the change is complete —
and before archive. Archive then merges the functional deltas into `openspec/specs/` and leaves
`standards/` untouched. Merge is the finish.

The one thing the gate canNOT do is block direct pushes to the default branch — that is your git
host's branch protection (see Governance). What the gate proves and doesn't prove is the honesty
note above: consistency + no-silent-regression, NOT spec truth.

---

## Tests, coverage, regression

**A test on every scenario.** A test is a PROJECTION of a scenario, regenerable from it; the
projection's FORM is the stack's choice. No scenario ships without a wired-in test carrying its
marker:

- **Functional scenario** → a test (unit / e2e / property / contract / type-level /
  runtime-assertion — whatever checks THAT scenario).
- **Standard @ mechanical** → a test asserting the enforcer (a unit/contract/type/runtime test, or
  a lint/AST rule — the stack's choice, named in the test body and the standard's prose) FLAGS the
  `violates` example and PASSES the `conforms` example (those scenarios ARE the fixtures).
- **Standard @ judge** → the registered, re-runnable judge check IS its test.

**Coverage** audits the CHANGE, not the whole pre-existing tree. It reads the functional-spec
deltas + the touched/created `standards/` files + test markers + git. Per touched functional spec
AND per touched/created standard, EVERY `#### Scenario:` must have ≥1 test carrying its `@spec
<capability-or-standard>/<requirement>/<scenario>` marker; a scenario with no projecting test is a
defect and MUST fail. Coverage runs PRE-archive (everything it reads is on disk). Coverage =
scenario→test linkage; strength = ratification. (A judge-tier standard counts as covered by its
registered judge check; coverage does NOT demand a mechanical test of a judge clause. A separate
`<coverage --all>` is a non-blocking backlog report that may scan everything.)

Add a **traceability marker** tying each test to the SCENARIO it projects — default `@spec
<capability-or-standard>/<requirement>/<scenario>` as a comment in the test file — so the coverage
check can grep it. A single test MAY carry MULTIPLE `@spec` markers, one per scenario it projects;
the marker sits on each test CASE (each `it()`/`test` block, or each RuleTester `valid`|`invalid`
fixture group). A `standard@mechanical` check-test covering `conforms` AND `violates` carries TWO
markers. Scenario names are matched like requirement names (case-sensitive, trailing-whitespace
normalized); renaming a scenario ⇒ re-project its marker.

**Regression** runs the FULL existing check/test suite (a change that greens its own checks but
reds an existing one is a regression and fails), AND diffs durable records against their
merge-base form:

- **Functional specs** — diff each touched spec's requirement+scenario set against its **BASELINE =
  the merge-base built spec** (`openspec/specs/<cap>/spec.md` as it stood before these deltas
  applied — **recovered from git, NOT from the archive delta**, which is only the
  ADDED/MODIFIED/REMOVED diff and cannot reconstruct the prior set). A requirement or scenario that
  disappeared without an explicit `## REMOVED Requirements` delta is a silent regression (archive's
  MODIFIED replaces the WHOLE requirement block) — fail.
- **Standards** — `standards/<name>.md` are plain git files; diff each touched standard against its
  git merge-base. A dropped requirement/scenario without a Rationale-log entry is a silent
  regression — fail.

A touched functional spec with NO built form at the merge-base (newly added) has no baseline —
coverage governs it instead, and the regression baseline is established from this change forward.

### Empirical OpenSpec 1.4.1 facts

These are tested, version-sensitive behaviors — re-verify on upgrade:

- **Delta-spec format (HARD).** Delta change specs use level-2 `## ADDED|MODIFIED|REMOVED
  Requirements`, level-3 `### Requirement: <name>`, level-4 `#### Scenario: <name>` (EXACTLY
  four hashes). A requirement body MUST contain literal `SHALL` or `MUST`. A built main spec
  needs `## Purpose` + `## Requirements` with ≥1 such requirement + ≥1 scenario. Renaming these
  tokens passes `schema validate` but silently breaks `validate`/`archive`.
- **`validate --strict`.** `## Purpose` < 50 chars fails ONLY under `--strict`. The structural
  rules (Purpose + Requirements present; each requirement a literal SHALL/MUST + ≥1 scenario) are
  HARD errors that fail even without `--strict`. `## Why` < 50 chars and > 10 deltas do NOT fail
  strict — if you want those bars, add them as your own grep/length sub-step, not as a `--strict`
  claim.
- **Archive merges functional deltas natively.** On a MODIFY archive, `openspec archive` rebuilds
  the built spec from the merge-base form + the delta; on a CREATE archive it builds from the
  ADDED delta body and stubs `## Purpose` as "TBD … Update Purpose after archive". Either way the
  REQUIREMENTS survive — a functional spec is a pure OpenSpec spec, so archive handles it natively.
  Filling in a stubbed Purpose after a CREATE is OpenSpec's OWN native step — you do it after
  archive.
- **`standards/` is never archived.** Top-level `standards/<name>.md` live OUTSIDE `openspec/`, so
  `openspec archive` never touches them — their frontmatter, Decision Record, and Rationale log are
  always intact. They are ordinary committed git files.
- **Archive matches requirement names CASE-SENSITIVELY**, normalizing trailing whitespace only —
  keep the traceability marker's casing exact, and bind the regression matcher to the same
  trailing-only, case-sensitive normalization.

### The openspec-instructions delivery mechanism

OpenSpec (verified on 1.4.1) emits, per artifact, a `<template>` block prefaced "Use this as the
structure for your output file. Fill in the sections" AND a separate `<instruction>` block ("AI
instructions for creating this artifact"). So the how-to GUIDANCE prose (the "Capabilities" specs
guidance, the "Standards loop" design guidance, the "Standard record" guidance, the "Enforcement
tasks/coverage/regression" tasks guidance, the proposal guidance) is set as the artifact's
`instruction:` field in the forked `schema.yaml` — NOT appended to the template file. The
`template:` files stay clean FILL-IN SCAFFOLDS; genuine document STRUCTURE (e.g. the proposal's
`## Affected surfaces & rollback` heading) MAY remain a small template addition.

`openspec update` writes only the **generic** skill/command wrappers from the installed CLI — they
carry NONE of your guidance. Your guidance binds the agent at **authoring time**: the wrappers call
`openspec instructions <artifact> --change <id>`, which emits both blocks (`instruction:` = your
guidance, `template:` = your scaffold). So the logic lives in the schema and is delivered on demand,
not baked into skills. Smoke-test it: `openspec new change probe && openspec instructions specs
--change probe` must print the "Capabilities" guidance. If it doesn't, the guidance isn't wired and
the schema is inert.

---

## Governance — make the flow the only path

Want "every change goes through OpenSpec" to be a *rule*, not a hope? It's just more standards.
`templates/governance.md` ships three seed standards (they follow `templates/standard.md` and drop
under top-level `standards/<name>.md` — or `standards/_org/` if shared):

- **changes-via-openspec** — the default branch accepts only archived OpenSpec changes; every
  non-generated file in the diff traces to that change's tasks/specs.
- **branch-per-change** — exactly one change per branch/worktree, named `change/<id>` (id ==
  branch suffix).
- **change-lifecycle** — gate-green + human-approved + archived before merge; the approval
  RATIFIES the record as the source of truth.

**The in-repo-vs-host split.** Governance is two halves, and both are required:

- **In repo (version-controlled):** the three governance standards above (each naming its enforcer
  in its test body), run by the gate (step 1). They key off `openspec/changes/`, the branch name,
  and the PR diff. Only the mechanical legs are in-repo-checkable: archive-present, single-change,
  name-match.
- **At the git host (cannot live in the repo):** **branch protection** on the default branch —
  no direct pushes, require a PR, require the gate's CI check green, require ≥1 approval. This is
  the half that makes "all changes via the flow" binding. Without it the rule is advisory; with
  it, binding. Record the exact host settings in `project.md`.

`changes-via-openspec` and `branch-per-change` are identical across projects — prime candidates
for the org-standards bundle, so every repo inherits the same immutable rule.

---

## Brownfield adoption

spec-first governs **changes**, not your pre-existing tree — so it installs green on a real repo
and ratchets coverage upward one change at a time. Coverage is **CHANGE-SCOPED**: an untouched
legacy spec is never audited; touch one → you spec it and test its scenarios ("you touched it, you
specify it"). Before the first change:

- **Scope strict to changed paths** (or bring the repo `validate --strict`-clean once) so an
  unrelated change never reds on legacy well-formedness.
- **Enforce-on-touch:** each change fully specs and tests what it adds or modifies. Coverage and
  `--strict` are change-scoped, never a whole-tree audit. A non-blocking `<coverage --all>` backlog
  report scans the whole tree so you can SEE the gap without it reding the build — the ratchet.

**Bootstrap standards from existing conventions.** On a fresh adoption, RETRIEVE finds nothing, so
every cross-cutting decision looks like ungoverned CREATE ground. Seed `standards/` from what you
ALREADY enforce: for each existing rule (eslint/biome config, tsconfig flags, CI checks,
CONTRIBUTING conventions) write a STANDARD whose enforcer RESOLVES to that existing tool (named in
its test body) — this is resolve-then-author documenting enforcement you already have, not new
machinery. Scope each `governs:` and bias to FEW. This gives design-time RETRIEVE something real to
find and turns implicit conventions into enforced, reviewable standards one at a time.

(Greenfield: skip all of this — there's no legacy tree, and `standards/` starts empty.)

## Org-shared standards

> **Secondary, optional capability.** The headline is that standards bind across the *whole
> project*; this is a bonus for orgs that want the *same* standard enforced in many repos.

The portable unit is the **standard FILE** — its rule + conforms/violates SCENARIOS — which is
stack-agnostic. You share the SCENARIO; each consuming project PROJECTS those scenarios into its
OWN enforcement form (its own test). Reuse is **copy-in seed**, the same shape as the schema bundle
itself (OpenSpec has no remote-URL referencing — [issue
#1131](https://github.com/Fission-AI/OpenSpec/issues/1131)):

- **Ship** a versioned `standards/` bundle: complete `standard.md` instances, each `governs:`-scoped,
  each carrying its Decision Record + rule + conforms/violates scenarios + Rationale log. (A shared
  config you also ship — e.g. a base eslint-config / tsconfig — is a convenient projection target,
  not the shared unit.)
- **Adopt** per project by copying them under the reserved prefix `standards/_org/<name>.md`,
  **pinning** the bundle version in `project.md`, and PROJECTING each standard's scenarios into a
  local test in the project's own enforcement form.
- **Make them mandatory** via the gate's org-standards integrity step (gate step 2): every
  `standards/_org/` file must match the pinned bundle on its **NORMALIZED** rule-bearing fields
  (frontmatter + the requirement/scenario set, ignoring whitespace/comments — raw byte-match flips
  on reformatting), so a locally loosened org rule reds the gate. The standard FILE (incl.
  scenarios) is shared and immutable; the projection/test is LOCAL. `standards/` is never archived,
  so the file stays exactly as shipped. To change one, **upstream a REVISE and bump the pin** — you
  cannot weaken it locally.

> **Reuse the FLOW vs reuse the RULES.** Dropping `spec-first/` in `~/.local/share` shares the
> *empty workflow*, not your standards — a standard is a `standards/` file *instance*, not a
> template, so it can't ride in `schema.yaml`. To share enforced rules, use the org-shared bundle
> above.

---

## Worked end-to-end example

`examples/add-rate-limit/` is a complete change walked through the whole flow, end to end. It
demonstrates: a **functional capability** (the rate limiter's behavior, a pure OpenSpec spec with
Given/When/Then scenarios each projected into a test); a **standard CREATE** caught by the design
standards loop on ungoverned ground (a cross-cutting rate-limit policy authored as a self-contained
`standards/<name>.md` — `governs:` scope, Decision Record, conforms/violates scenarios that double
as the test's fixtures, Rationale log); the **tasks** that resolve-then-author each enforcer, test,
and the per-scenario `@spec <capability-or-standard>/<requirement>/<scenario>` marker; the **gate**
run continuously and green as the last group — PRE-archive; and **archive as the finish** (the
functional delta merges into `openspec/specs/`, the `standards/` file is left untouched). It also
shows **ratification** — the standard CREATE is surfaced at PR review (a mechanical check flags the
`standards/` change in the diff; the human reviewer ratifies it; this change carries no judge-tier
standard, so the local judge runs in the loop and finds none). Read it alongside this README to see
every section above in one concrete change.

## What's inside

```
.
├── README.md                       # this file
├── LICENSE
├── openspec/schemas/spec-first/
│   ├── schema.yaml                 # full schema shape: artifacts + conditional `standard` + optional `judge`
│   ├── SETUP.md                    # step-by-step install
│   └── templates/
│       ├── _additions.md           # the guidance prose → set as each artifact's `instruction:` in schema.yaml (proposal/specs/design/standard/tasks)
│       ├── standard.md             # the standards/<name>.md template (governs+tier + ADR + rule + conforms/violates + rationale log)
│       ├── gate.md                 # the gate block, embedded by project.md
│       ├── governance.md           # seed standards: changes-via-openspec, branch-per-change, change-lifecycle
│       ├── project.md              # the project contract: gate + org-pin + branch-protection + adoption
│       └── judge.md                # the judge template (a LOCAL in-loop reviewer, hard-local; CI is mechanical; the human ratifies)
└── examples/
    └── add-rate-limit/             # a complete change walked end-to-end through the flow
```

`schema.yaml` declares the base artifacts (`proposal` → `{specs, design}`), the **conditional
`standard`** artifact (`requires: [design]`, `generates: ../../../standards/*.md` — TOP-LEVEL,
outside `openspec/`, authored only on CREATE/REVISE), `tasks` (`requires: [specs, design,
standard]`), and the optional `judge` artifact (`requires: [tasks]`). `standard.md`, `gate.md`,
`governance.md`, and `project.md` are **reference templates** — `standard.md` is the
`standards/<name>.md` scaffold the `standard` artifact generates; `gate.md`, `governance.md`, and
`project.md` are layered in at setup.

## Install

This is an **overlay**, not a from-scratch schema: it builds on a **fork of the built-in
`spec-driven` schema**, so it inherits the delta-spec format (`ADDED` / `MODIFIED` / `REMOVED`
requirements) that `openspec archive` and `openspec sync` rely on. That's why install starts with
a fork.

1. In your project: `openspec schema fork spec-driven spec-first` — writes
   `openspec/schemas/spec-first/` (project-local, version-controlled).
2. Set the guidance blocks in `templates/_additions.md` as the artifact `instruction:` fields in
   your forked `schema.yaml` (`proposal` / `specs` / `design` / `standard` / `tasks`) — NOT
   appended to the template files. The `template:` files stay clean fill-in scaffolds; add only the
   small `## Affected surfaces & rollback` structure to the proposal template. (Additive per-team
   CONSTRAINTS — e.g. "this team's proposals must include a rollback plan" — belong in
   `config.rules` per-artifact, distinct from the universal flow in `instruction:`/templates.)
3. Add the **conditional `standard` artifact** to your forked `schema.yaml`: it generates to
   `../../../standards/*.md` (top-level `<repo>/standards/`, OUTSIDE `openspec/` — so archive never
   touches it), `requires: [design]`, and is a no-op unless the design loop flags CREATE/REVISE.
   Point `tasks` at `requires: [specs, design, standard]`. Copy `templates/standard.md` (the
   `standards/<name>.md` scaffold the artifact fills), `templates/judge.md`, `templates/gate.md`,
   `templates/governance.md`, and `templates/project.md` into the schema's `templates/`, and add
   the `standard` and `judge` rows to `schema.yaml` (keep the fork's other rows — they carry the
   right `template:` pointers and the `apply:` block). `gate.md`, `governance.md`, and `project.md`
   are reference templates — no `schema.yaml` row. (Reconcile each `template:` pointer with the
   on-disk filename — a fork may emit `spec.md`; mismatch fails `schema validate`.)
4. `openspec schema validate spec-first`.
5. In `openspec/config.yaml`, set `schema: spec-first` + `context: <your stack>`, then `openspec
   update`. **Note:** `openspec update` writes only the *generic* skill/command wrappers — it does
   NOT bake your guidance into them. That guidance binds the agent at authoring time via `openspec
   instructions <artifact> --change <id>` (the per-artifact `<instruction>` block), which those
   wrappers call. **Smoke-test:** `openspec new change probe && openspec instructions specs
   --change probe` should print the "Capabilities" guidance.

Then **define the gate** (required — this is what makes the flow bite): paste `templates/gate.md`
into `openspec/project.md` and wire its single command (the steps). And, recommended, **adopt
governance** + set **branch protection** at your host. Full detail in
`openspec/schemas/spec-first/SETUP.md`.

## The project.md contract

Everything stack-specific lives in `openspec/project.md` — the ONE file where a project meets the
flow. It carries **four things** the schema templates reference abstractly:

1. **The gate** — command, the gate steps, and the traceability marker (paste from `gate.md`).
2. **The org-standards pin** — bundle + version the integrity step matches `standards/_org/`
   against, or `none`.
3. **The branch-protection settings** — the host rules the gate cannot enforce (no direct pushes,
   PR required, gate-green required, ≥1 approval). Recorded so the guarantee is auditable.
4. **The adoption mode** — greenfield | brownfield, strict scope, and the change-scoped-coverage
   ratchet note.

Copy `templates/project.md` and fill the `<…>` slots; it embeds `gate.md`. The schema templates
stay stack-agnostic; this one file is where a project meets the flow.

## Cross-project / team use

- **All your own projects:** drop the `spec-first/` folder in
  `~/.local/share/openspec/schemas/` (user-global) — it's then available everywhere, one copy of
  the flow.
- **A team:** this repo *is* the distribution. Teammates clone/copy the folder in. OpenSpec has
  **no remote-URL schema referencing** (you can't point a project at a GitHub schema by URL — see
  [issue #1131](https://github.com/Fission-AI/OpenSpec/issues/1131)); the bundle is brought local,
  the same way every community schema works.

## Versioning & dogfooding

Tested against **OpenSpec 1.4.1**; archive behavior and `validate --strict` behavior are
version-sensitive — re-verify on upgrade. Upgrades are manual copy-in: diff `templates/` and
re-run `openspec schema validate`. And honestly: **this repo is a schema *bundle*, not itself a
spec-first project** — it ships the flow, it doesn't run under it (a template distribution has no
runtime or test suite to gate).
