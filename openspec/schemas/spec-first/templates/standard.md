# Standard (non-functional capability) — complete template

A STANDARD is a cross-cutting rule the whole project obeys: a coding convention, an
architectural decision, a security/perf/observability bar. It is an ORDINARY spec under
`specs/` — same OpenSpec flow, same delta format — made cross-cutting by `governs:`.

This file is the complete, **archive-safe** shape. On a MODIFY archive, `openspec archive`
rebuilds the `## Requirements` section and DELETES anything inside it that is not a `### Requirement:`
block, so everything else (frontmatter, the ADR, the rationale-log) lives ABOVE
`## Requirements` or in a trailing `## ` section. Copy the body below into
`specs/<standard-name>/spec.md` and fill the angle-bracket slots.

> **CREATE-vs-MODIFY caveat (tested on OpenSpec 1.4.1).** A spec's FIRST archive (CREATE)
> rebuilds the main spec from the ADDED delta ALONE — it DROPS the frontmatter CORE, the
> Decision Record, and the Rationale log, and stubs `## Purpose` as "TBD - created by archiving".
> Only on later MODIFY archives does that content survive. So after a standard's first archive,
> RE-APPLY its frontmatter CORE + ADR + rationale log to `openspec/specs/<name>/spec.md`, fill
> `## Purpose`, and re-run the gate — that frontmatter is the `test:` CORE the gate's coverage
> keys on. This re-apply is also a GATE SUB-CHECK: after a new spec's CREATE archive the gate
> FAILS if the built spec is missing its CORE (`test:`), so a forgotten re-apply cannot silently
> pass coverage. Version-sensitive — re-verify on upgrade.

---

```
---
test:       <ref to the test(s) PROJECTED from this standard's scenarios — the coverage key, MANDATORY. The scenarios are canonical; the test is their regenerable projection (re-project when a scenario changes). mechanical: a check that PROJECTS the conforms/violates scenarios — the enforcer FLAGS the `violates` fixture and PASSES the `conforms` fixture; judge: the registered judge run. No resolvable test ⇒ coverage fails ⇒ gate red. The enforcer's FORM is stack-specific — a unit/e2e/property/contract test, a type-level check, a runtime assertion, OR a lint/AST rule (lint is just ONE option, not the default) — and is NAMED in this test body and the prose below, NOT a frontmatter field.>
tier:       mechanical | judge        # machine-checkable-at-all, NOT the mechanism. mechanical = a machine can decide it (the projection is an executable test / type check / runtime assertion / lint/AST rule — whatever the stack uses; hard). judge = needs-human-judgment, focused LLM review (hard locally, soft in CI). NO advisory tier. GATE-VERIFIED: the gate asserts this tier matches what `test:` resolves to and fails on mismatch.
governs:    <scope — the surfaces/paths/layers this rule applies to; this is what makes it cross-cutting. Its PRESENCE marks this spec a standard.>
adoption:   legacy        # OPTIONAL. Presence exempts a PRE-EXISTING spec from coverage until it is next modified (then the exemption drops — "you touched it, you specify it"). OMIT on active/new specs.
---

# <Standard name>

## Purpose
<Why this standard exists — one short paragraph; keep ≥ 50 chars or `validate --strict`
fails (a strict-only check in 1.4.1). State what breaks across the project without it.>

## Decision Record
<!-- ADR — include when this standard encodes an ARCHITECTURAL decision. Above
     ## Requirements so archive preserves it. Omit the section for a plain convention. -->
- **Status:** proposed | accepted | superseded by <link>
- **Context:** <the forces / constraints / problem that made a decision necessary>
- **Decision:** <what we decided, stated affirmatively>
- **Consequences:** <what gets easier; what gets harder; the trade-off accepted>
- **Alternatives considered:** <option → why rejected; one line each>

## Requirements

### Requirement: <the rule, named>
The system SHALL <the normative clause — MUST contain literal SHALL or MUST>.

#### Scenario: conforms
- **Given** <a compliant situation>
- **When** the enforcer runs
- **Then** it passes.

#### Scenario: violates
- **Given** <a non-compliant situation — this is the fixture the projected `test:` asserts the enforcer flags>
- **When** the enforcer runs
- **Then** it fails, pointing the author at the fix.

## Rationale log
<!-- Trailing section — survives archive. Accretes; never rewrite history. -->
- **v1 (<date>):** <why the rule is shaped this way>.
- **v2 (<date>):** <each REVISE — a loosening / carve-out / tightening — appends a dated,
  justified entry here. This is the standard's amendment history (the ADR's living tail).>
```

---

## Notes

- **Why the clause is a real `### Requirement:`.** Only a requirement with SHALL/MUST + a
  scenario is visible to the spec contract and survives archive. The `### Requirement:`
  SHALL/MUST clause IS the authoritative one-liner — there is no separate `invariant:` field.
  The conforms/violates scenarios are not decoration — each scenario is the canonical,
  stack-agnostic statement of the property, and its `test:` is a regenerable PROJECTION of it.
  The conforms/violates scenarios are the PROJECTED FIXTURES the enforcer runs against; re-project
  the test when a scenario changes. That is what makes "a test on every standard" concrete.
- **The enforcer lives in `test:`, not its own field.** For a `mechanical` standard the enforcer
  is named in the test body and the prose above — there is no `check:` frontmatter field — and its
  FORM is the stack's choice: an executable test (unit / e2e / property / contract), a type-level
  check, a runtime assertion/contract, OR a lint/AST rule id (e.g.
  `eslint-plugin-local/require-rate-limit`). A behavioral scenario typically projects to an
  EXECUTABLE test; a purely structural rule MAY project to a lint/AST check — but lint is one
  option, not the default. The gate's coverage keys on `test:` alone.
- **The scenario is the traceable unit (the `@spec` marker).** Mark each projected test with
  `@spec <capability>/<requirement>/<scenario>` — the scenario, not just the requirement, is the
  generative unit. A single test MAY carry MULTIPLE markers, one per scenario it projects: a
  standard@mechanical check covering BOTH conforms AND violates carries TWO markers (natural
  placement is on each test case / each valid|invalid fixture group). Coverage verifies, per
  touched spec, that EVERY `#### Scenario:` has ≥1 test carrying its marker — a scenario with no
  projecting test is declared-but-unprojected and FAILS. This proves each scenario is LINKED to a
  wired-in test that runs; it does NOT prove the test faithfully exercises the scenario (an empty
  test with the right marker passes) — test STRENGTH is ratification's job. Scenario names match
  like requirement names (case-sensitive, trailing-whitespace normalized); renaming a scenario ⇒
  re-project its marker.
- **Functional capabilities** use the forked base `specs` template shape (interface +
  Given/When/Then scenarios). They carry the same frontmatter CORE minus `governs:` (its absence
  is what marks them functional); their `test:` points at the behavioral tests realizing their
  scenarios.
- **`binding` is derived, never declared.** A clause binds IFF its `test:` resolves — there
  is no `binding:` field. The coverage check operationalizes this; don't add a field for it.
- **Authoring aid.** The `engineering:architecture` skill is a good way to think through the
  Decision Record, but reshape its output into this layout (ADR above `## Requirements`).
