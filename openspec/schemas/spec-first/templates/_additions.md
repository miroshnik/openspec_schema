# Guidance for the forked artifacts

These blocks are the spec-first FLOW. After forking, each one is set as the matching
artifact's **`instruction:` field** in `openspec/schemas/spec-first/schema.yaml` (the
`<instruction>` block OpenSpec emits per artifact — "AI instructions for creating this
artifact" — NOT the `<template>` block). The forked `template:` files stay clean
FILL-IN scaffolds; only genuine document STRUCTURE (the proposal's
`## Affected surfaces & rollback` heading) is a small template addition. The labels below
say which artifact each block belongs to.

> **Why these are prose, not config.** OpenSpec core has no native
> `governs:` / standards / coverage / tests-per-spec / gate primitive — `validate`
> and `archive` never read them (issues #900 and #829 are unshipped). Everything
> here is therefore a PAIR: (a) instruction the agent follows, plus (b) an external
> mechanical check the project's **gate** runs (see `templates/gate.md`).
> `openspec validate --strict` is only ONE input to that gate.

> **Archive-safety (read before placing anything).** On a MODIFY archive (the steady state),
> `openspec archive` rebuilds each spec file: it keeps everything ABOVE `## Requirements` and any trailing
> `## ` section, and DELETES anything inside `## Requirements` that is not a
> `### Requirement:` block. So custom frontmatter, the ADR, and the rationale-log
> go ABOVE `## Requirements` or in a trailing section — never between requirements.
> Never rename the hard-coded tokens (`## ADDED|MODIFIED|REMOVED|RENAMED
> Requirements`, `### Requirement:`, `#### Scenario:`, literal `SHALL`/`MUST`):
> renaming them passes `schema validate` but silently breaks `validate`/`archive`.

> **CREATE archive drops non-requirement content (tested on 1.4.1).** A spec's FIRST archive
> rebuilds the main spec from the ADDED delta ALONE — the frontmatter CORE, the ADR, and the
> rationale log are DROPPED and `## Purpose` is stubbed "TBD". They survive only on later
> MODIFY archives. So after a new spec's first archive, RE-APPLY its frontmatter/ADR/rationale
> to `openspec/specs/<name>/spec.md` and re-fill Purpose (the tasks block has the step, and the
> gate backstops a forgotten re-apply — see that block).

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
## Capabilities & standards (spec-first)

Every spec under specs/ is a capability. Two shapes, ONE mechanism — distinguished
by a `governs:` field in the frontmatter. Both shapes obey the SAME hard contract
every spec has: `## Purpose` + `## Requirements`, ≥1 requirement carrying SHALL/MUST,
≥1 `#### Scenario:`. (See templates/standard.md for the complete, archive-safe shape.)

A test is a PROJECTION of a scenario, REGENERABLE from it: the scenario (Given/When/Then,
or conforms/violates) is the canonical, STACK-AGNOSTIC statement of the property; the test
is its LOCAL artifact. When a scenario changes, RE-PROJECT its test.

FUNCTIONAL capability = a component's behavior.
  Body: typed interface (provides/requires) + behavioral scenarios (Given/When/Then).
  Its TEST = the test(s) PROJECTING its scenarios — typically executable tests.
  No `governs:` field — its ABSENCE marks the spec functional.

STANDARD = non-functional capability = a cross-cutting rule. Add `governs:` (the
surfaces it applies to) — its PRESENCE marks the spec a standard. A standard is just as
much a spec — same delta format, same flow, lives under specs/ — it simply binds across
the whole project, not one component.
  Body: the rule as a real `### Requirement:` (SHALL/MUST + a conforms scenario AND a
  violates scenario) + conforms/violates examples (these double as the test's fixtures).
  If the standard encodes an ARCHITECTURAL decision, add a `## Decision Record` (ADR)
  section ABOVE `## Requirements` — Status / Context / Decision / Consequences /
  Alternatives considered — and a trailing `## Rationale log` that accretes v1/v2.
  The ADR lives HERE, in the standard's spec, not in a separate document.
  Its TEST = the test PROJECTING its conforms/violates scenarios, or the registered judge run.

Shared machine-read CORE — keep these in FRONTMATTER (above ## Purpose, so archive
preserves them), fixed place, so the gate and design-retrieval find them regardless
of body shape. These are spec-first CONVENTIONS, not OpenSpec fields (verified on 1.4.1:
`validate --strict` returns valid, 0 issues, on a spec carrying this CORE):
  test:       <ref to the test(s) that enforce/prove this spec — the coverage key; MANDATORY>
  tier:       mechanical | judge        # GATE-VERIFIED against what test: resolves to
  governs:    <scope>                   # STANDARDS ONLY — presence marks a standard; absent ⇒ functional
  adoption:   legacy                    # OPTIONAL — presence exempts a pre-existing spec from
                                        #   coverage until first modified; OMIT on active specs
There is no `check:`, `invariant:`, or `binding:` field. For a standard@mechanical the
ENFORCER — whatever the stack uses to check that scenario: a unit / e2e / property-based /
contract test, a type-level check, a runtime assertion, OR a lint/AST rule (lint is just one
option, not THE form) — is named in the test/check body and the spec prose, NOT in frontmatter;
the gate's coverage keys on `test:` only. The `### Requirement:` SHALL/MUST clause IS the
authoritative one-liner; there is no separate invariant field (a functional spec naming a
cross-scenario property states it in prose). And binding is DERIVED — a clause binds IFF its
test resolves.

TIERS — machine-checkable-at-all vs needs-human-judgment, NOT a mechanism. Two, no escape hatch:
  mechanical — checkable by machine with no human in the loop. The hard-binding tier;
               mechanize first. The scenario's NATURE hints at the projection CLASS
               (behavioral → an executable test; purely structural → a static-analysis
               check), but the concrete tool is the STACK'S choice — not necessarily lint.
  judge      — focused per-standard LLM review of the part a machine cannot catch
               (intent / naming / ergonomic fit). REQUIRED to RUN and SURFACE (cannot be
               skipped), but it does NOT self-enforce: hard locally (blocks closing the task),
               soft in CI (posts suspects), BINDING only when a human ratifies its suspects at
               PR review. Every judge clause MUST register a runnable judge check.
The gate VERIFIES the declared tier matches what `test:` resolves to (any machine-checkable
test/check artifact ⇒ mechanical; a registered judge run ⇒ judge) and FAILS on mismatch.

A TEST ON EVERY SPEC. Functional → tests projecting its scenarios. Standard@mechanical →
a test asserting the enforcer FLAGS the violates example and PASSES the conforms example.
Standard@judge → the registered, re-runnable judge check IS its test. No spec ships
without a resolvable `test:`.

AUTHORING STANDARDS. If design's standards loop flagged a CREATE or REVISE, author the
standard spec / delta HERE — specs is where standards live; design only decided. Use
templates/standard.md for the archive-safe shape (ADR + rationale log in the right place).
```

---

## → set as the design artifact `instruction:` field

```
## Standards loop (spec-first)

Before writing the design, and whenever a cross-cutting decision arises:

RETRIEVE — pull the standards whose `governs:` matches the surfaces this change
touches (from the proposal). Apply them. Retrieval is first-try efficiency; the
gate is the backstop for misses.

Then for each cross-cutting decision:
  COMPLY  — fits an existing standard → follow it.
  CREATE  — hits ungoverned ground → IDENTIFY the need for a new standard and specify its
            requirement / tier / test (and, if it is an architectural call, its Decision
            Record). design DECIDES; specs AUTHORS the standard spec — record the decision
            here, don't write specs/ — writing the standard in design would invert #0 (a
            derivative, the recorded decision, preceding its source spec). specs is its
            authoritative home. Bias to FEW — a standard taxes every future change.
  REVISE  — in tension with an existing standard → decide the DELTA (what loosens /
            tightens / carves out) and its justification; specs writes the delta and
            appends the standard's Rationale log (v2). Distinguish comply-vs-revise
            honestly; loosening needs strong justification.

RATIFICATION — every CREATE / REVISE is surfaced for human sign-off at PR review: the
soft-CI judge posts "standards-changed" suspects and a human approves the PR. Do not
self-merge a standards change — standards bind the whole project; never let one land
agent-only.

ENFORCEMENT — for each clause, RECORD whether its test will be REUSED (an existing
project test already covers it) or newly AUTHORED. The full resolve-then-author mechanics
live in tasks; design just records the call.
```

---

## → set as the tasks artifact `instruction:` field

```
## Enforcement tasks (spec-first)

SOURCE-FIRST (#0): the spec is the source of truth; the check, the test, the code, and any
migration are PROJECTIONS of it. To change behavior, amend the spec/scenario FIRST (as a delta),
then re-project its test and code — never edit a derivative behind a scenario you didn't touch
(the regression diff won't always catch it; it's a discipline the author owns and the reviewer
ratifies).

For every binding clause introduced or touched (a functional capability or a standard):
  - RESOLVE its ENFORCER against existing project checks; if none fits, AUTHOR one at the
    correct TIER (machine-checkable-at-all vs needs-human-judgment, not a mechanism):
      mechanical — checkable by machine with no human in the loop; the hard-binding tier,
                   mechanize first. The PROJECTION FORM is the stack's choice — a unit /
                   e2e / property-based / contract test, a type-level check, a runtime
                   assertion, or a lint/AST rule (lint is one option, not the default).
                   The scenario's nature hints at the class (behavioral → executable test;
                   purely structural → static analysis); the concrete tool is stack-specific.
      judge      — focused per-standard review of the un-mechanizable part
                   (intent / naming / ergonomics); hard locally, soft in CI.
  - RESOLVE or AUTHOR its TEST as a PROJECTION of the spec's scenarios: functional → tests
    projecting its scenarios; standard@mechanical → a test that the enforcer flags the
    violates example and passes the conforms example; standard@judge → register the runnable
    judge check. Add a traceability marker tying each test back to the SCENARIO it projects
    (default `@spec <capability>/<requirement>/<scenario>` on each test case — each it()/test
    block or RuleTester fixture group). A single test MAY carry SEVERAL markers, one per
    scenario it projects; a standard@mechanical check-test covering conforms AND violates
    carries TWO. (Name the mechanical enforcer in the test/spec prose, not in frontmatter —
    the gate keys coverage on `test:` only.)
  - Declare each spec's `tier:` honestly: the gate VERIFIES it against what `test:` resolves
    to and FAILS on mismatch (any machine-checkable artifact must say mechanical; a judge run,
    judge).
  - If the change breaks an existing standard's contract, add the MIGRATION — itself
    a projection of the change (codemod / data-migration / compat shim).

COVERAGE: for every spec TOUCHED BY THIS CHANGE (added/modified), its `test:` MUST resolve to
a real, wired-in test AND every `#### Scenario:` of that spec MUST have ≥1 test carrying its
`@spec <capability>/<requirement>/<scenario>` marker. A spec whose `test:` does not resolve, OR
a scenario with no projecting test (declared-but-unprojected), is a defect, not a state — it MUST
fail. This scenario-linkage is the faithful operationalization of "tests are projections of
scenarios" (stricter than the old requirement-level linkage). (Coverage keys on `test:` only;
scenario names are matched like requirement names — case-sensitive, trailing-whitespace
normalized — so renaming a scenario means re-projecting its marker. It counts a judge clause as
covered by its registered judge check; it does NOT demand a mechanical test of a judge clause. A
spec marked `adoption: legacy` is exempt until first modified — then the exemption drops. Coverage
audits the CHANGE, not the whole pre-existing tree; a separate non-blocking backlog report can scan
everything.)

HONESTY: coverage proves each SCENARIO is LINKED to a wired-in test that runs — it does NOT prove
the test faithfully or non-trivially exercises the scenario (an empty test with the right marker
passes). Test STRENGTH is ratification's job.

REGRESSION: the gate runs the FULL existing check/test suite, not only this change's checks
— a change that greens its own checks but reds an existing one is a regression and fails.
It also diffs each touched main spec's requirement+scenario set against its BASELINE = the
full built spec at this change's merge-base (openspec/specs/<cap>/spec.md as it stood before
these deltas were applied — recover it from git, NOT from the archive delta, which is only
the ADDED/MODIFIED/REMOVED diff and cannot reconstruct the prior set). A requirement or
scenario that disappeared without an explicit `## REMOVED Requirements` delta is a silent
regression (archive's MODIFIED replaces the WHOLE requirement block) — fail.

POST-ARCHIVE RE-APPLY (new specs): a spec's FIRST archive rebuilds it from the ADDED delta and
DROPS its frontmatter CORE, ADR, and rationale log, stubbing Purpose "TBD" (OpenSpec 1.4.1). For
every newly-created capability/standard, add a task to RE-APPLY that content to
openspec/specs/<name>/spec.md and re-run the gate after first archive — without the CORE (`test:`)
the coverage has nothing to read. This human step is now BACKSTOPPED by a gate sub-check: after a
NEW spec's first (CREATE) archive the gate FAILS if the built spec is missing its CORE, so a
forgotten re-apply cannot silently pass coverage.

GATE, CONTINUOUSLY: run the project's GATE (defined in project.md; see templates/gate.md)
after each task group during the change — not only at the end — so breakage surfaces early.
LAST GROUP = the full gate green; it MUST be green before the change is complete. The gate
aggregates: every project mechanical check + the coverage check + the regression check +
`openspec validate --strict` + the judge (hard locally, soft in CI). New functional
capabilities add their tests + the traceability marker so coverage sees they're covered.
```
