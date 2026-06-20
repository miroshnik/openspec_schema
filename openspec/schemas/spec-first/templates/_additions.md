# Guidance for the forked artifacts

These blocks ARE the instruction: values. After forking, each one is set as the matching
artifact's **`instruction:` field** in `openspec/schemas/spec-first/schema.yaml` (the
`<instruction>` block OpenSpec emits per artifact — "AI instructions for creating this
artifact" — NOT the `<template>` block). The forked `template:` files stay clean FILL-IN
scaffolds; only genuine document STRUCTURE (the proposal's `## Affected surfaces & rollback`
heading) is a small template addition. The labels below say which artifact each block belongs to.

> **Why these are prose, not config.** OpenSpec core has no native
> `governs:` / standards / coverage / tests-per-spec / gate primitive — `validate`
> and `archive` never read them (issues #900 and #829 are unshipped). Everything
> here is therefore a PAIR: (a) instruction the agent follows, plus (b) an external
> mechanical check the project's **gate** runs (see `templates/gate.md`).
> `openspec validate --strict` is only ONE input to that gate.

> **Where durable spec-first content lives (read before placing anything).** Durable content
> lives OUTSIDE the change folder AND OUTSIDE `openspec/specs/`. Two homes, two shapes:
>   - **FUNCTIONAL capability** → a PURE native OpenSpec spec at `openspec/specs/<cap>/spec.md`
>     (`## Purpose` + `## Requirements` with `### Requirement:` SHALL/MUST + `#### Scenario:`).
>     It survives archive NATIVELY because requirements are exactly what archive keeps.
>   - **STANDARD** → a self-contained living file at top-level `standards/<name>.md` (a committed
>     git file OUTSIDE `openspec/` that archive does not touch). Everything about it — scope, tier,
>     decision record, the rule, its scenarios, amendment history — lives in that one file. See
>     `templates/standard.md`.
> Never rename the hard-coded tokens (`## ADDED|MODIFIED|REMOVED|RENAMED Requirements`,
> `### Requirement:`, `#### Scenario:`, literal `SHALL`/`MUST`): renaming them passes
> `schema validate` but silently breaks `validate`/`archive`.

---

## → set as the proposal artifact `instruction:` field

```
RETRIEVABILITY. Make the change retrievable by the design's standards loop: name the
surfaces it touches so design can pull the standards whose `governs:` scope covers them,
and state the rollback path. This is the one block with a matching template STRUCTURE —
the `## Affected surfaces & rollback` heading lives in the proposal TEMPLATE; fill it.
```

The one genuine template-structure addition (add this heading to the forked `proposal.md`):

```
## Affected surfaces & rollback
- List the surfaces this change touches — design uses this to retrieve the
  standards whose `governs:` scope covers them.
- State the rollback path.
```

---

## → set as the specs artifact `instruction:` field

```
## Functional capabilities (spec-first)

Specs under specs/ are FUNCTIONAL capabilities — a component's behavior. A functional
capability is a PURE native OpenSpec spec — the contract every OpenSpec spec already has:
  ## Purpose       — one short paragraph (keep >= 50 chars or `validate --strict` fails).
  ## Requirements  — >=1 `### Requirement:` carrying literal SHALL/MUST, each with >=1
                     `#### Scenario:` (Given/When/Then).
Because requirements are exactly what archive keeps, a functional capability survives
`openspec archive` NATIVELY: its `## ADDED Requirements` delta rebuilds the main spec on archive.

A test is a PROJECTION of a scenario, REGENERABLE from it: the scenario (Given/When/Then)
is the canonical, STACK-AGNOSTIC statement of the behavior; the test is its LOCAL artifact.
When a scenario changes, RE-PROJECT its test. The test ties back to the scenario with a
scenario-level marker — `@spec <capability>/<requirement>/<scenario>`, one per scenario a
test projects (a test may carry several). That marker is the coverage key.

A brand-new functional spec is archived with OpenSpec's own stubbed `## Purpose` ("TBD ...
Update Purpose after archive"). Filling it in is OpenSpec's OWN native behavior — you do it
after archive, as part of finishing the spec.

CROSS-CUTTING RULES are NOT specs. If design's standards loop flagged a CREATE or REVISE,
the rule is authored as a STANDARD — a self-contained file at top-level standards/<name>.md,
NOT under openspec/specs/. See templates/standard.md and the standard artifact. specs/ holds
only functional capabilities here.
```

---

## → set as the design artifact `instruction:` field

```
## Standards loop (spec-first)

Before writing the design, and whenever a cross-cutting decision arises:

RETRIEVE — read standards/ directly and pull the records whose `governs:` scope matches
the surfaces this change touches (from the proposal). Apply them. Retrieval is first-try
efficiency; the gate is the backstop for misses.

Then for each cross-cutting decision:
  COMPLY  — fits an existing standard → follow it. No new standards/ file.
  CREATE  — hits ungoverned ground → DECIDE the new standard: its scope, its tier
            (mechanical | judge), the normative rule, and — because every standard is an
            architectural record — its Decision Record. design DECIDES; the `standard`
            artifact AUTHORS the file at standards/<name>.md. Record the decision HERE;
            don't write standards/ from design (that would invert #0 — a derivative, the
            recorded decision, preceding the standard it authors). Bias to FEW — a standard
            taxes every future change.
  REVISE  — in tension with an existing standard → decide the DELTA (what loosens /
            tightens / carves out) and its justification; the standard artifact edits
            standards/<name>.md in place and appends its Rationale log. Distinguish
            comply-vs-revise honestly; loosening needs strong justification.
  DEFER   — looks cross-cutting, but bias-to-FEW says it's too early to standardize: one
            surface, not clearly a class yet. KEEP it LOCAL — it stays the functional
            capability's behavior (its scenarios). RECORD it as a standard CANDIDATE: a line
            in the proposal or design. PROMOTE it later via CREATE + a migration when a second
            surface needs the same rule, OR when a reviewer flags it. A standard is a rule that
            has OUTGROWN one capability; until then it lives as spec behavior. You do not
            standardize on a hunch.

The standards loop is CONDITIONAL: the standard artifact runs only on a CREATE or REVISE.
A change that only COMPLIES or DEFERS touches no standards/ file.

RATIFICATION — every CREATE / REVISE is surfaced for human sign-off at PR review: a
mechanical check flags the standards change in the diff (git diff touches standards/) so
the human reviewer ratifies it at PR review. No LLM does this surfacing — it is
deterministic. Do not self-merge a standards change — standards bind the whole project;
never let one land agent-only.

ENFORCEMENT — for each clause, RECORD whether its test will be REUSED (an existing
project test already covers it) or newly AUTHORED. The full resolve-then-author mechanics
live in tasks; design just records the call.
```

---

## → set as the standard artifact `instruction:` field

```
## Standard (spec-first) — a living record outside the change

Author this ONLY when design's standards loop flagged a CREATE (new file) or REVISE (edit
in place). Otherwise it's a no-op — no new standards/ file.

A STANDARD is a SELF-CONTAINED living file at top-level standards/<name>.md. It is NOT an
OpenSpec spec, lives OUTSIDE openspec/, and is NEVER touched by `openspec archive` — so its
frontmatter and prose are always safe and always visible. See templates/standard.md for the
complete shape; in brief:
  --- (frontmatter)  governs: <scope> ; tier: mechanical | judge ; ---
  # <Standard name>
  ## Decision Record   — ADR: Status / Context / Decision / Consequences / Alternatives.
  ## Rule
  ### Requirement: <name>  → The system SHALL <normative clause; literal SHALL/MUST>.
  #### Scenario: conforms ...     #### Scenario: violates ...
  ## Rationale log   — a dated entry on why the rule is shaped this way; each REVISE appends a
                       dated, justified entry (the amendment history).

Everything about the standard lives HERE, always visible. RETRIEVE reads standards/ directly;
the gate reads its scenarios for coverage. Frontmatter is safe because the file is never
archived. Standards are NOT validated by `openspec validate` (they are spec-first artifacts,
not openspec specs) — the GATE validates them: governs + tier present, a Rule with a
`### Requirement:` SHALL/MUST clause, plus a `conforms` scenario AND a `violates` scenario.

TIER — machine-checkable-at-all vs needs-human-judgment, NOT a mechanism. Two, no escape hatch:
  mechanical — a machine can decide it with no human in the loop. The hard-binding tier;
               mechanize first. The scenario's NATURE hints at the projection CLASS
               (behavioral → an executable test; purely structural → a static-analysis
               check), but the concrete tool is the STACK'S choice — not necessarily lint.
  judge      — focused per-standard LLM review of the part a machine cannot catch
               (intent / naming / ergonomic fit). A LOCAL in-loop reviewer: REQUIRED to RUN
               and SURFACE in the authoring loop; hard-local (blocks closing the task). It is
               NOT a CI job — CI is mechanical-only. BINDING only when a human ratifies its
               suspects at PR review.
The gate VERIFIES the declared tier matches what the conforms/violates test resolves to (any
machine-checkable test/check ⇒ mechanical; a registered judge run ⇒ judge) and FAILS on mismatch.
```

---

## → set as the tasks artifact `instruction:` field

```
## Enforcement tasks (spec-first)

ONE PASS. Everything happens here in a single pass: author/fill the functional specs, author
or edit the standards/ records, project the tests, resolve-or-author the checks, add any
migration. There is no later phase: by the time tasks finish, the CHANGE holds everything the
gate needs.

SOURCE-FIRST (#0): the spec/standard is the source of truth; the check, the test, the code,
and any migration are PROJECTIONS of it. To change behavior, amend the spec/scenario (or the
standard's rule) FIRST, then re-project its test and code — never edit a derivative behind a
scenario you didn't touch (the regression diff won't always catch it; it's a discipline the
author owns and the reviewer ratifies).

For every binding clause introduced or touched (a functional capability scenario or a
standard's rule):
  - RESOLVE its ENFORCER against existing project checks; if none fits, AUTHOR one at the
    correct TIER (machine-checkable-at-all vs needs-human-judgment, not a mechanism):
      mechanical — checkable by machine with no human in the loop; the hard-binding tier,
                   mechanize first. The PROJECTION FORM is the stack's choice — a unit /
                   e2e / property-based / contract test, a type-level check, a runtime
                   assertion, or a lint/AST rule (lint is one option, not the default).
                   The scenario's nature hints at the class (behavioral → executable test;
                   purely structural → static analysis); the concrete tool is stack-specific.
      judge      — focused per-standard review of the un-mechanizable part
                   (intent / naming / ergonomics); a LOCAL in-loop reviewer, hard-local
                   (blocks closing the task). CI is mechanical-only; the human ratifies at review.
  - RESOLVE or AUTHOR its TEST as a PROJECTION of the scenarios: functional → tests projecting
    its scenarios; standard → a test that the enforcer FLAGS the `violates` example and PASSES
    the `conforms` example; standard@judge → register the runnable judge check. Add a
    traceability marker tying each test back to the SCENARIO it projects:
    `@spec <capability-or-standard>/<requirement>/<scenario>` on each test case (each it()/test
    block or RuleTester fixture group). A single test MAY carry SEVERAL markers, one per scenario
    it projects; a standard check covering conforms AND violates carries TWO.
  - Declare each standard's `tier:` honestly: the gate VERIFIES it against what the test
    resolves to and FAILS on mismatch (any machine-checkable artifact must say mechanical; a
    judge run, judge).
  - If the change breaks an existing standard's contract, add the MIGRATION — itself
    a projection of the change (codemod / data-migration / compat shim).

COVERAGE (scenario-level): for every touched FUNCTIONAL spec AND every touched/created
STANDARD, EVERY `#### Scenario:` MUST have >=1 test carrying its
`@spec <cap-or-standard>/<requirement>/<scenario>` marker. A scenario with no projecting test
(declared-but-unprojected) is a defect, not a state — it MUST fail. Coverage reads the CHANGE
(functional-spec deltas) + the standards/ files + the test markers + git. It runs PRE-archive —
everything exists by then. (Scenario names are matched like requirement names — case-sensitive,
trailing-whitespace normalized — so renaming a scenario means re-projecting its marker. A judge
clause counts as covered by its registered judge check; coverage does NOT demand a mechanical
test of a judge clause. Coverage is CHANGE-SCOPED: untouched specs are not audited; touch
one and you must spec+test it. A separate non-blocking `coverage --all` backlog report can scan
everything — the brownfield ratchet.)

HONESTY: coverage proves each SCENARIO is LINKED to a wired-in test that runs — it does NOT
prove the test faithfully or non-trivially exercises the scenario (an empty test with the right
marker passes). Test STRENGTH is ratification's job.

REGRESSION (via git): the gate runs the FULL existing check/test suite, not only this change's
checks — a change that greens its own checks but reds an existing one is a regression and fails.
It also diffs against the git MERGE-BASE: each touched functional spec's requirement+scenario set
against the spec as it stood before this change (recover it from git, NOT from the archive delta,
which is only the ADDED/MODIFIED/REMOVED diff and cannot reconstruct the prior set), and the
standards/ files diffed in git. A requirement or scenario that disappeared without an explicit
`## REMOVED Requirements` delta is a silent regression — fail. Git is the prior-state source,
so the check runs PRE-archive.

GATE, CONTINUOUSLY: run the project's GATE (defined in project.md; see templates/gate.md) after
each task group during the change — not only at the end — so breakage surfaces early. The LAST
TASK GROUP = run the GATE GREEN. It MUST be green before the change is complete. The gate reads
the CHANGE: the functional-spec deltas + the standards/ files + the tests + the full suite + git
— everything exists PRE-archive. The gate is MECHANICAL — it aggregates: every project mechanical
check + the coverage check + the regression check + `openspec validate --strict` (functional specs
only; standards are gate-validated, not openspec-validated). This mechanical gate is the required
check CI runs and what you run locally after each task group; NO LLM is in it. The judge is a
SEPARATE LOCAL step in the authoring loop (hard-local), not part of the CI gate.

ARCHIVE + MERGE are the post-green FINISH, NOT a task group — no work follows the green gate.
`openspec archive` merges the functional-spec deltas into openspec/specs/ (requirements survive
natively) and moves the change to changes/archive/. The standards/ files are committed git files,
untouched by archive. Merge is the finish.
```
