# spec-first — an OpenSpec schema

**Principle #0 — the spec is the single source of truth.** Code, tests, checks, migrations,
and docs are all *projections* of specs: a test projects a scenario — the scenario (Given/When/Then,
or conforms/violates) is the canonical, stack-agnostic statement of the property, and the test is
its local, regenerable artifact (when the scenario changes, RE-PROJECT its test). The projection's
FORM is stack-specific — a unit / e2e / property / contract / type-level / runtime-assertion test,
or a lint/AST rule — whatever that stack uses to check THAT scenario. A migration projects the
change, code is born in `apply` from tasks (which project from specs). On any divergence the spec
is authoritative and changes FIRST — you never alter a derivative behind its spec's back.
Everything else in this README — the two spec shapes, the frontmatter CORE, the flow order, the
per-spec `test:`, the gate, the all-via-flow governance — exists to enforce that one rule.

> **An OpenSpec flow with checks you can trust your generation against, plus standards that
> are mandatory across the whole project — enforced, not filed in a wiki.**

---

## Table of contents

1. [The problem](#the-problem)
2. [What the gate does and doesn't prove (honesty)](#what-the-gate-does-and-doesnt-prove)
3. [The model — two shapes, one mechanism](#the-model--two-shapes-one-mechanism)
4. [The flow & lifecycle](#the-flow--lifecycle)
5. [The standards loop](#the-standards-loop)
6. [The gate](#the-gate)
7. [Tests on every spec, coverage, regression](#tests-on-every-spec-coverage-regression)
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

- **Functional behavior** *and* **cross-cutting standards** live as specs under `specs/`.
- **Every spec carries a mandatory `test:`** — the single enforcement ref: each scenario projects
  to a test whose FORM is the stack's choice (unit / e2e / property / contract / type-level /
  runtime-assertion test, or a lint/AST rule) — a behavioral scenario typically to an executable
  test, a purely structural rule MAY project to a lint/AST check.
- A single **gate** must be green before a change completes (full suite + org-integrity +
  coverage + regression + `validate --strict` + judge).
- What no machine can check, an **LLM judge** reviews — hard locally, soft in CI (it never
  reds a shared build).
- A **standard** is a cross-cutting rule that binds every surface it governs across the *whole
  project* — enforced by the gate, not filed in a wiki. (It can optionally be shared across
  projects too — a bonus, below — but that is not the point.)

Adoption is incremental: spec-first governs **changes**, not your existing tree — the gate
ratchets coverage upward one change at a time and stays green throughout. It is a **community
schema** — an overlay you fork into your project (built on OpenSpec's `spec-driven`), so it
inherits the delta-spec format that `archive` / `sync` rely on.

## What the gate does and doesn't prove

This is the honest line, and it is the whole point of the design:

**Green means** spec, test, and code are CONSISTENT, and that no scenario silently regressed.
It makes the spec the thing you *review*. It does NOT prove the spec is correct or its test
strong: the same agent may author spec, scenario, check, and test, so a hollow self-consistent
spec also goes green. The spec becomes trustworthy only when a human **ratifies** it at PR
review. The gate makes divergence-from-the-spec mechanical; **ratification** makes the spec
itself authoritative.

So the trust split is:

| What | Who/what guarantees it |
| --- | --- |
| Projections are consistent with the spec | the gate (mechanical) |
| No scenario silently disappeared | the gate (regression diff) |
| Every scenario is actually projected | the gate (coverage) |
| The un-mechanizable part fits the spirit | the judge (surfaces; human ratifies) |
| **The spec itself is true / its tests are strong** | **a human, at PR approval** |

Coverage proves each SCENARIO is LINKED to a marked, wired-in test that RUNS — it cannot prove the
test faithfully or non-trivially exercises the scenario (an empty `it()` with the right `@spec`
marker passes). It is linkage, not strength. Test *strength*, like spec *truth*, is what
ratification at PR review checks.

---

## The model — two shapes, one mechanism

One mechanism, **two shapes of spec**, distinguished by a single frontmatter field, `governs:`:

- **Functional capability** — a component's behavior. Body: typed interface (provides/requires)
  + behavioral scenarios (Given/When/Then). The `### Requirement:` SHALL/MUST clauses ARE the
  authoritative properties (a cross-scenario property, if named, lives in prose — not frontmatter).
  **Its scenarios become its tests.** No `governs:`.
- **Standard** (`governs:` present) — a cross-cutting rule the whole project obeys (a coding
  convention, a security/perf/observability bar, an architectural decision). It is an *ordinary*
  spec under `specs/` — same delta format, same flow — made cross-cutting by `governs:`. Body:
  the rule as a real `### Requirement:` (SHALL/MUST + a **conforms** scenario AND a **violates**
  scenario) + conforms/violates examples (which double as the test fixtures).
  Architectural standards additionally carry a full **ADR** (`## Decision Record`: Status /
  Context / Decision / Consequences / Alternatives) plus a trailing `## Rationale log` that
  accretes across revisions. **Its conforms/violates scenarios projected into a test (mechanical)
  or its registered judge run (judge) is its `test:`** — the concrete enforcer (a unit/contract/
  type/runtime test, or a lint/AST rule, whatever the stack uses to check the rule) is named in
  the test body and the prose, not in frontmatter.

Both shapes obey the SAME hard OpenSpec contract every spec has: `## Purpose` + `## Requirements`,
≥1 requirement carrying literal `SHALL`/`MUST`, ≥1 `#### Scenario:`.

### The frontmatter CORE

Both shapes carry the same machine-read CORE in **frontmatter** (above `## Purpose`, so archive
preserves it; fixed place so the gate and design-retrieval find it regardless of body shape):

```
test:       <ref to the test(s) that project this spec's scenarios — the coverage key>  # MANDATORY
tier:       mechanical | judge          # machine-checkable vs needs-judgment; gate-VERIFIED against what test: resolves to
governs:    <scope>                     # standards only; PRESENCE marks a standard, ABSENT ⇒ functional capability
adoption:   legacy                      # OPTIONAL — presence exempts a pre-existing spec from coverage until first modified; OMIT on active specs
```

Just four fields. `test:` is the SINGLE enforcement ref — the coverage key the gate resolves; for
a `standard@mechanical` the concrete enforcer (a unit/contract/type/runtime test, or a lint/AST
rule id like `eslint-plugin-local/require-rate-limit` — the stack's choice) is named in the test
body and the spec prose, NOT a frontmatter field. There is no `check:` (folded into `test:`), no
`invariant:` (the `### Requirement:`
SHALL/MUST clause IS the authoritative one-liner), and no `binding:` (derived). `governs:` is the
discriminator: its PRESENCE marks a standard; ABSENT ⇒ functional capability. `adoption:` is now
presence-of-`legacy` (omit ⇒ active), not an enum.

These are spec-first **conventions**, not native OpenSpec fields — `validate`/`archive` never
read them (verified on 1.4.1: a spec carrying the CORE returns `valid:true`, 0 issues under
`--strict`). Verify your forked `validate` treats unknown frontmatter as ignorable, or the gate
reds on `validate --strict` for every spec.

**`tier` is GATE-VERIFIED, `binding` is DERIVED — neither is taken on trust.** `tier` is
machine-checkable-at-all (mechanical) vs needs-human-judgment (judge) — it is NOT the mechanism;
the scenario's NATURE hints at the projection CLASS (behavioral → executable test; structural →
static analysis), but the concrete tool is stack-specific. The gate asserts the declared `tier`
matches what `test:` resolves to (a machine-checkable test artifact ⇒ mechanical; a registered
judge run ⇒ judge) and FAILS on mismatch — the same discipline as "binding is derived." A clause
binds IFF its `test:` resolves; there is no `binding:` field, the coverage check operationalizes
it.

### Tiers — two, no escape hatch

- **mechanical** — AST / lint / type / test. The hard-binding tier. Mechanize first.
- **judge** — focused per-standard LLM review of the part a machine cannot catch (intent /
  naming / ergonomic fit). REQUIRED to RUN and SURFACE (cannot be skipped), but it does NOT
  self-enforce: **hard locally** (blocks closing the task), **soft in CI** (posts suspects),
  BINDING only when a human ratifies its suspects at PR review. Every judge clause MUST register
  a runnable judge check.

There is no advisory tier. Each rule is a PAIR: the authoring guidance the agent follows (set as
the artifact's `instruction:` field in the forked schema) + an external mechanical check the
project's **gate** runs. OpenSpec has no native `governs:`/coverage/test-per-spec/gate primitive,
which is exactly why the teeth are external.

---

## The flow & lifecycle

A change moves through the OpenSpec artifact DAG:

```
proposal → specs → design → tasks → apply → (judge)
```

- **proposal** — what changes and why; plus **Affected surfaces & rollback** (design uses the
  surfaces to retrieve the standards whose `governs:` covers them).
- **specs** — author functional capabilities AND standards. This is where standards LIVE. If
  design's loop flagged a CREATE or REVISE, the standard spec / delta is authored *here*.
- **design** — the technical design + the **standards loop** (below). design DECIDES standards;
  specs AUTHORS them — writing a standard in design would invert #0 (a derivative, the recorded
  decision, preceding its source spec).
- **tasks** — enforcer @ tier + test-per-spec + migration + coverage + regression + the continuous
  gate. **SOURCE-FIRST discipline (#0):** to change behavior, amend the spec/scenario FIRST (as
  a delta), then re-project its test and code — never edit a derivative behind a scenario you
  didn't touch.
- **apply** — implement the tasks (inherited from the fork; tracks `tasks.md`). Code is born here.
- **judge** — runs POST-apply (it judges the implemented diff). Optional artifact; re-runnable.

The full git lifecycle (from `templates/governance.md`):

```
0. (brownfield) adoption mode set — see Brownfield adoption.
1. START   — openspec new change <id>; branch: git worktree add ../<id> -b change/<id>
2. AUTHOR  — proposal → specs → design → tasks. design DECIDES standards; specs AUTHORS them.
3. BUILD   — implement tasks; run the GATE after each task group (continuously) until green,
             resolving/justifying every hard-local judge suspect.
4. ARCHIVE — openspec archive <id> → merges delta specs into openspec/specs/, moves the change
             to changes/archive/. Re-run the gate over the updated specs until green.
5. PR      — open the PR from change/<id>. CI runs the gate (soft judge posts suspects).
6. REVIEW  — a human approves, ratifying any standards CREATE/REVISE (and the spec as source of truth).
7. MERGE   — squash-merge to the default branch. Direct pushes are blocked at the host.
```

### The standards loop

Before writing the design, and whenever a cross-cutting decision arises:

- **RETRIEVE** — pull the standards whose `governs:` matches the surfaces this change touches
  (from the proposal). Apply them. Retrieval is first-try efficiency; the gate is the backstop
  for misses.

Then for each cross-cutting decision:

- **COMPLY** — fits an existing standard → follow it.
- **CREATE** — hits ungoverned ground → IDENTIFY the need for a new standard and specify its
  requirement / enforcer / tier / test (and, if architectural, its Decision Record). design
  DECIDES; specs AUTHORS the standard spec. Bias to FEW — a standard taxes every future change.
- **REVISE** — in tension with an existing standard → decide the DELTA (what loosens / tightens /
  carves out) and its justification; specs writes the delta and appends the standard's Rationale
  log (v2). Distinguish comply-vs-revise honestly; loosening needs strong justification.

**RESOLVE-THEN-AUTHOR.** For each clause, RECORD whether its check and test will be REUSED (an
existing project check/test already covers it) or newly AUTHORED. Resolve against existing
project checks first; only author when none fits.

**RATIFICATION.** Every CREATE / REVISE is surfaced for human sign-off at PR review: the soft-CI
judge posts "standards-changed" suspects and a human approves the PR. Never self-merge a
standards change — standards bind the whole project.

---

## The gate

spec-first references "the project's **gate**" abstractly so the schema stays stack-agnostic.
The gate is **EXTERNAL** to OpenSpec — `openspec validate --strict` is only one of its inputs.
It is a SINGLE command that both CI and the local authoring loop run: **continuously during a
change (after each task group), not only at the end**, and required green as the LAST task group
before the change is complete.

It runs **six steps**, fails on the first red for steps 1–5, and runs the judge last:

1. **Project mechanical checks** — `<lint/type/test command>`. The FULL existing suite, not only
   this change's checks. (Governance standards' enforcer commands run here too, like any standard
   — no extra step.)
2. **Org-standards integrity** — `<integrity command>` (skip if you consume no org bundle). Every
   spec under `specs/_org/` MUST match the pinned org-standards bundle on its **NORMALIZED**
   rule-bearing fields (frontmatter `test`/`tier`/`governs` + the requirement/scenario set,
   ignoring whitespace/comments — raw byte-match flips on reformatting). A locally loosened org
   standard is a forbidden divergence — fail.
3. **Coverage** — `<coverage command>`. For every spec TOUCHED BY THIS CHANGE (added/modified
   delta): its `test:` MUST resolve to a real, wired-in test, AND every `#### Scenario:` in it MUST
   have ≥1 test carrying that scenario's `@spec <capability>/<requirement>/<scenario>` marker
   (coverage keys on `test:` only — there is no `check:` to resolve). A scenario with no projecting
   test ⇒ declared-but-unprojected ⇒ fail. The gate also asserts the declared `tier` matches what
   `test:` resolves to (machine-checkable test ⇒ mechanical; registered judge run ⇒ judge) and
   FAILS on mismatch. A judge-tier clause is covered by its registered judge check, not a
   mechanical test. Untouched specs are not re-audited; `adoption: legacy` specs are exempt until
   first modified. **Re-apply-after-CREATE is a gate sub-check:** after a new spec's first (CREATE)
   archive the gate FAILS if the built spec is missing its CORE (`test:`), so a forgotten re-apply
   cannot silently pass coverage.
4. **Regression diff** — `<regression command>`. For each touched main spec, diff its
   requirement+scenario set against its BASELINE = the **full built spec at this change's
   merge-base** (recovered from git, NOT from the archive delta). A requirement or scenario that
   disappeared without an explicit `## REMOVED Requirements` (or `## RENAMED Requirements`) delta
   is a silent regression — fail.
5. **`openspec validate --strict`** — exit 0. (Structural rules are HARD errors even without
   `--strict`; `## Purpose` < 50 chars fails only under `--strict` — see the 1.4.1 facts below.)
6. **Judge** (tier: judge clauses) — `<judge command>`: an agent in print mode reviewing the
   governed scope for the part no mechanical check covers.
   - **Locally:** BLOCKING. Do not close a task while the judge has an `open` suspect — fix it,
     or mark it `justified` with a pointer to the spec's Rationale log.
   - **In CI:** SOFT. Post suspects as PR annotations / a "human-review-needed" flag. NEVER
     auto-fail the build on a non-deterministic verdict.

**Green-as-last-group.** The gate runs throughout the change so breakage surfaces early; the
FINAL task group is "the full gate green," and it MUST be green before the change is complete.

The one thing the gate canNOT do is block direct pushes to the default branch — that is your git
host's branch protection (see Governance). What the gate proves and doesn't prove is the honesty
note above: consistency + no-silent-regression, NOT spec truth.

---

## Tests on every spec, coverage, regression

**A test on every spec.** `test:` is the SINGLE enforcement ref — the one frontmatter field the
coverage check resolves. A test is a PROJECTION of a scenario, regenerable from it; the projection's
FORM is the stack's choice. No spec ships without it resolving:

- **Functional** → each scenario projects to a test (unit / e2e / property / contract / type-level /
  runtime-assertion — whatever checks THAT scenario).
- **Standard @ mechanical** → a test asserting the enforcer (a unit/contract/type/runtime test, or a
  lint/AST rule — the stack's choice, named in the test body and the spec prose) FLAGS the
  `violates` example and PASSES the `conforms` example (those scenarios ARE the fixtures).
- **Standard @ judge** → the registered, re-runnable judge check IS its test.

**Coverage** audits the CHANGE, not the whole pre-existing tree. Per touched spec (added/modified),
EVERY `#### Scenario:` must have ≥1 test carrying its `@spec
<capability>/<requirement>/<scenario>` marker; a scenario with no projecting test is a defect and
MUST fail. This is stricter than the old requirement-level linkage and is the faithful
operationalization of "tests are projections of scenarios." (A judge clause counts as covered by
its registered judge check; coverage does NOT demand a mechanical test of a judge clause.
`adoption: legacy` specs are exempt until first modified. A separate `<coverage --all>` is a
non-blocking backlog report that may scan everything.)

Add a **traceability marker** tying each test to the SCENARIO it projects — default `@spec
<capability>/<requirement>/<scenario>` as a comment in the test file — so the coverage check can
grep it. A single test MAY carry MULTIPLE `@spec` markers, one per scenario it projects; the
marker sits on each test CASE (each `it()`/`test` block, or each RuleTester `valid`|`invalid`
fixture group). A `standard@mechanical` check-test covering `conforms` AND `violates` carries TWO
markers. Scenario names are matched like requirement names (case-sensitive, trailing-whitespace
normalized); renaming a scenario ⇒ re-project its marker.

**Regression** runs the FULL existing check/test suite (a change that greens its own checks but
reds an existing one is a regression and fails), AND diffs each touched main spec's
requirement+scenario set against its **BASELINE = the merge-base built spec**
(`openspec/specs/<cap>/spec.md` as it stood before these deltas applied — **recovered from git,
NOT from the archive delta**, which is only the ADDED/MODIFIED/REMOVED diff and cannot reconstruct
the prior set). A requirement or scenario that disappeared without an explicit `## REMOVED
Requirements` delta is a silent regression (archive's MODIFIED replaces the WHOLE requirement
block) — fail. A touched spec with NO built form at the merge-base (newly added, or the first
modify of an `adoption: legacy` spec) has no baseline — coverage governs it instead, and the
regression baseline is established from this change forward.

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
- **Archive rebuilds full-file (MODIFY steady-state).** On a MODIFY archive against an existing
  built spec, `openspec archive` keeps everything ABOVE `## Requirements` and any trailing `## `
  section, and DELETES anything inside `## Requirements` that is not a `### Requirement:` block —
  so custom frontmatter, the ADR, and the rationale log live ABOVE `## Requirements` or in a
  trailing section, never between requirements. The FIRST/CREATE archive is the exception (next
  bullet): it builds from the ADDED delta body ALONE, carrying it verbatim and dropping CORE/ADR/
  rationale — so keep CREATE deltas minimal and re-apply after.
- **Archive-on-CREATE drops non-requirement content (the re-apply caveat).** A spec's FIRST
  archive (CREATE) rebuilds the main spec from the ADDED delta ALONE — it DROPS the frontmatter
  CORE, the ADR, and the rationale log, and stubs `## Purpose` as "TBD". Only later MODIFY
  archives preserve that content. **So after a new spec's first archive, RE-APPLY its
  frontmatter CORE (`test:`) + ADR + rationale log to `openspec/specs/<name>/spec.md` and re-fill
  Purpose, then re-run the gate** — without the CORE, coverage has nothing to read. This re-apply
  is BACKSTOPPED by the gate (a CREATE that lands without its CORE FAILS coverage; see step 3), so
  a forgotten re-apply cannot silently pass. (The tasks block carries the human-facing step.)
- **Archive matches requirement names CASE-SENSITIVELY**, normalizing trailing whitespace only —
  keep the traceability marker's casing exact, and bind the regression matcher to the same
  trailing-only, case-sensitive normalization.

### The openspec-instructions delivery mechanism

OpenSpec (verified on 1.4.1) emits, per artifact, a `<template>` block prefaced "Use this as the
structure for your output file. Fill in the sections" AND a separate `<instruction>` block ("AI
instructions for creating this artifact"). So the how-to GUIDANCE prose (the "Capabilities &
standards" specs guidance, the "Standards loop" design guidance, the "Enforcement
tasks/coverage/regression/re-apply" tasks guidance, the proposal guidance) is set as the artifact's
`instruction:` field in the forked `schema.yaml` — NOT appended to the template file. The
`template:` files stay clean FILL-IN SCAFFOLDS; genuine document STRUCTURE (e.g. the proposal's
`## Affected surfaces & rollback` heading) MAY remain a small template addition.

`openspec update` writes only the **generic** skill/command wrappers from the installed CLI — they
carry NONE of your guidance. Your guidance binds the agent at **authoring time**: the wrappers call
`openspec instructions <artifact> --change <id>`, which emits both blocks (`instruction:` = your
guidance, `template:` = your scaffold). So the logic lives in the schema and is delivered on demand,
not baked into skills. Smoke-test it: `openspec new change probe && openspec instructions specs
--change probe` must print the "Capabilities & standards" guidance. If it doesn't, the guidance
isn't wired and the schema is inert.

---

## Governance — make the flow the only path

Want "every change goes through OpenSpec" to be a *rule*, not a hope? It's just more standards.
`templates/governance.md` ships three seed standards (they follow `templates/standard.md`, drop
under `specs/governance/<name>/spec.md` or `specs/_org/`):

- **changes-via-openspec** — the default branch accepts only archived OpenSpec changes; every
  non-generated file in the diff traces to that change's tasks/specs.
- **branch-per-change** — exactly one change per branch/worktree, named `change/<id>` (id ==
  branch suffix).
- **change-lifecycle** — gate-green + human-approved + archived before merge; the approval
  RATIFIES the spec as the source of truth.

**The in-repo-vs-host split.** Governance is two halves, and both are required:

- **In repo (version-controlled):** the three governance enforcers above (each named in its
  standard's `test:` body), run by the gate (step 1). They key off `changes/`, the branch name,
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
and ratchets coverage upward one change at a time. The `adoption:` field is the legacy ratchet.
Before the first change:

- **Scope strict to changed paths** (or bring the repo `validate --strict`-clean once) so an
  unrelated change never reds on legacy well-formedness.
- **Tag every pre-existing spec `adoption: legacy`** — `legacy` exempts it from the coverage
  check until it is next modified, when the exemption drops ("you touched it, you specify it").
- Then **enforce-on-touch**: each change fully specs and tests what it adds or modifies. Coverage
  and `--strict` are **change-scoped**, never a whole-tree audit.

**Bootstrap standards from existing conventions.** On a fresh adoption, RETRIEVE finds nothing, so
every cross-cutting decision looks like ungoverned CREATE ground. Seed the corpus from what you
ALREADY enforce: for each existing rule (eslint/biome config, tsconfig flags, CI checks,
CONTRIBUTING conventions) write a STANDARD whose `test:` RESOLVES to that existing tool (the
enforcer named in its body) — this is resolve-then-author documenting enforcement you already have,
not new machinery. Mark each `adoption: legacy`, scope its `governs:`, and bias to FEW. This gives
design-time RETRIEVE something real to find and turns implicit conventions into enforced, reviewable
standards one at a time.

(Greenfield: skip all of this — there's no legacy tree, so `adoption:` is omitted everywhere.)

## Org-shared standards

> **Secondary, optional capability.** The headline is that standards bind across the *whole
> project*; this is a bonus for orgs that want the *same* standard enforced in many repos.

The portable unit is the **standard's SPEC** — its rule + conforms/violates SCENARIOS — which is
stack-agnostic. You share the SCENARIO; each consuming project PROJECTS those scenarios into its
OWN enforcement form (its own test). Reuse is **copy-in seed**, the same shape as the schema bundle
itself (OpenSpec has no remote-URL referencing — [issue
#1131](https://github.com/Fission-AI/OpenSpec/issues/1131)):

- **Ship** a versioned `org-standards/` bundle: complete `standard.md` instances, each
  `governs:`-scoped, each carrying its rule + conforms/violates scenarios. (A shared config you
  also ship — e.g. a base eslint-config / tsconfig — is a convenient projection target, not the
  shared unit.)
- **Adopt** per project by copying them under the reserved prefix `specs/_org/<name>/spec.md`,
  **pinning** the bundle version in `project.md`, and PROJECTING each standard's scenarios into a
  local test in the project's own enforcement form.
- **Make them mandatory** via the gate's org-standards integrity step (gate step 2): every
  `specs/_org/` spec must match the pinned bundle on its **NORMALIZED** rule-bearing fields
  (frontmatter + the requirement/scenario set, ignoring whitespace/comments — raw byte-match
  flips on reformatting), so a locally loosened org rule reds the gate. The SPEC (incl. scenarios)
  is shared and immutable; the projection/test is LOCAL. `specs/_org/` is never a local delta
  target, so archive never rewrites it. To change one, **upstream a REVISE and bump the pin** —
  you cannot weaken it locally.

> **Reuse the FLOW vs reuse the RULES.** Dropping `spec-first/` in `~/.local/share` shares the
> *empty workflow*, not your standards — a standard is a spec *instance*, not a template, so it
> can't ride in `schema.yaml`. To share enforced rules, use the org-shared library above.

---

## Worked end-to-end example

`examples/add-rate-limit/` is a complete change walked through the whole flow, end to end. It
demonstrates: a **functional capability** (the rate limiter's behavior, with Given/When/Then
scenarios each projected into a test); a **standard CREATE** caught by the design standards loop on
ungoverned ground (a cross-cutting rate-limit policy with a `governs:` scope, an ADR, and
conforms/violates scenarios that double as the test's fixtures); the **frontmatter CORE** on
both shapes; the **tasks** that resolve-then-author each enforcer, test, and the per-scenario
`@spec <capability>/<requirement>/<scenario>` marker;
the **gate** run continuously and green as the last group; the **archive-on-CREATE re-apply**
caveat for the newly created specs; and **ratification** — the standard CREATE is surfaced at PR
review (the soft-CI judge posts a "standards-changed" suspect; this change carries no judge-tier
clause, so the judge step runs and finds none). Read it alongside this README to see every section
above in one concrete change.

## What's inside

```
.
├── README.md                       # this file
├── LICENSE
├── openspec/schemas/spec-first/
│   ├── schema.yaml                 # full schema shape: 4 base artifacts + optional `judge` + apply
│   ├── SETUP.md                    # step-by-step install
│   └── templates/
│       ├── _additions.md           # the guidance prose → set as each artifact's `instruction:` in schema.yaml (proposal/specs/design/tasks)
│       ├── standard.md             # complete archive-safe shape for a STANDARD spec (+ ADR + rationale log)
│       ├── gate.md                 # the 6-step gate block, embedded by project.md
│       ├── governance.md           # seed standards: changes-via-openspec, branch-per-change, change-lifecycle
│       ├── project.md              # the project contract: gate + org-pin + branch-protection + adoption
│       └── judge.md                # the judge template (hard local, soft CI)
└── examples/
    └── add-rate-limit/             # a complete change walked end-to-end through the flow
```

`schema.yaml` declares four base artifacts (`proposal` → `specs` → `design` → `tasks`), the
optional `judge` artifact (`requires: [tasks]`, runs post-apply), and the inherited `apply:`
block. `standard.md`, `gate.md`, `governance.md`, and `project.md` are **reference templates** —
layered in at setup, no `schema.yaml` row; only `judge.md` gets one.

## Install

This is an **overlay**, not a from-scratch schema: it builds on a **fork of the built-in
`spec-driven` schema**, so it inherits the delta-spec format (`ADDED` / `MODIFIED` / `REMOVED`
requirements) that `openspec archive` and `openspec sync` rely on. That's why install starts with
a fork.

1. In your project: `openspec schema fork spec-driven spec-first` — writes
   `openspec/schemas/spec-first/` (project-local, version-controlled).
2. Set the guidance blocks in `templates/_additions.md` as the artifact `instruction:` fields in
   your forked `schema.yaml` (`proposal` / `specs` / `design` / `tasks`) — NOT appended to the
   template files. The `template:` files stay clean fill-in scaffolds; add only the small
   `## Affected surfaces & rollback` structure to the proposal template. (Additive per-team
   CONSTRAINTS — e.g. "this team's proposals must include a rollback plan" — belong in
   `config.rules` per-artifact, distinct from the universal flow in `instruction:`/templates.)
3. Copy `templates/judge.md`, `templates/standard.md`, `templates/gate.md`,
   `templates/governance.md`, and `templates/project.md` into the schema's `templates/`, and add
   ONLY the `judge` row to your forked `schema.yaml` (keep the fork's other rows — they carry the
   right `template:` pointers and the `apply:` block). `standard.md`, `gate.md`, `governance.md`,
   and `project.md` are reference templates — no `schema.yaml` row. (Reconcile each `template:`
   pointer with the on-disk filename — a fork may emit `spec.md`; mismatch fails `schema validate`.)
4. `openspec schema validate spec-first`.
5. In `openspec/config.yaml`, set `schema: spec-first` + `context: <your stack>`, then `openspec
   update`. **Note:** `openspec update` writes only the *generic* skill/command wrappers — it does
   NOT bake your guidance into them. That guidance binds the agent at authoring time via `openspec
   instructions <artifact> --change <id>` (the per-artifact `<instruction>` block), which those
   wrappers call. **Smoke-test:** `openspec new change probe && openspec instructions specs
   --change probe` should print the "Capabilities & standards" guidance.

Then **define the gate** (required — this is what makes the flow bite): paste `templates/gate.md`
into `openspec/project.md` and wire its single command (the 6 steps). And, recommended, **adopt
governance** + set **branch protection** at your host. Full detail in
`openspec/schemas/spec-first/SETUP.md`.

## The project.md contract

Everything stack-specific lives in `openspec/project.md` — the ONE file where a project meets the
flow. It carries **four things** the schema templates reference abstractly:

1. **The gate** — command, the 6 checks, and the traceability marker (paste from `gate.md`).
2. **The org-standards pin** — bundle + version the integrity step matches `specs/_org/` against,
   or `none`.
3. **The branch-protection settings** — the host rules the gate cannot enforce (no direct pushes,
   PR required, gate-green required, ≥1 approval). Recorded so the guarantee is auditable.
4. **The adoption mode** — greenfield | brownfield, strict scope, and the legacy-ratchet note.

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

Tested against **OpenSpec 1.4.1**; archive-safety and `validate --strict` behavior are
version-sensitive — re-verify on upgrade. Upgrades are manual copy-in: diff `templates/` and
re-run `openspec schema validate`. And honestly: **this repo is a schema *bundle*, not itself a
spec-first project** — it ships the flow, it doesn't run under it (a template distribution has no
runtime or test suite to gate).
