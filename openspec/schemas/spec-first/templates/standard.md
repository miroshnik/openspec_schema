# Standard — the self-contained `standards/<name>.md` template

A STANDARD is a cross-cutting rule the whole project obeys: a coding convention, an
architectural decision, a security/perf/observability bar. A standard is a rule that has
OUTGROWN one capability. Unlike a functional capability, it is **NOT an OpenSpec spec** — it
is a self-contained, living RECORD that lives at top-level `standards/<name>.md` (a sibling of
`openspec/`, OUTSIDE `openspec/specs/`), generated/updated via the `standard` artifact
(`generates: ../../../standards/*.md`).

It is a committed git file. `openspec archive` does not touch it: the `governs`/`tier`
frontmatter, the Decision Record, the rule, and the rationale log are plain git content that is
committed and survives in place. Everything about the standard lives HERE, always visible —
scope, tier, why, the rule, its scenarios, its amendment history.

Standards are CONDITIONAL: author/update one only when design's standards loop flags a
CREATE or REVISE; otherwise this is a no-op (no new `standards/` file). `openspec validate`
does not validate standards — **the gate** validates them: `governs:` + `tier:` present, a
`### Requirement:` with literal SHALL/MUST, and BOTH a `conforms` and a `violates` scenario.
RETRIEVE (the design standards loop) reads `standards/` directly; the gate reads each
standard's scenarios for coverage.

Copy the body below into `standards/<name>.md` and fill the angle-bracket slots.

---

```
---
governs:    <scope — the surfaces/paths/layers this rule applies to; this is what makes it cross-cutting>
tier:       mechanical | judge        # machine-checkable-at-all, NOT the mechanism. mechanical = a machine can decide it (the projection is an executable test / type check / runtime assertion / lint/AST rule — whatever the stack uses; hard). judge = needs-human-judgment, a LOCAL in-loop review (hard-local; CI is mechanical, the human ratifies). NO advisory tier. GATE-VERIFIED: the gate asserts this tier matches what the rule's test resolves to and fails on mismatch.
---

# <Standard name>

## Decision Record
<!-- The ADR — the cross-cutting decision this standard encodes. Omit only for the rare
     convention with no decision behind it. -->
- **Status:** proposed | accepted | superseded by <link>
- **Context:** <the forces / constraints / problem that made a decision necessary>
- **Decision:** <what we decided, stated affirmatively>
- **Consequences:** <what gets easier; what gets harder; the trade-off accepted>
- **Alternatives considered:** <option → why rejected; one line each>

## Rule

### Requirement: <the rule, named>
The system SHALL <the normative clause — MUST contain literal SHALL or MUST>.

#### Scenario: conforms
- **Given** <a compliant situation>
- **When** the enforcer runs
- **Then** it passes.

#### Scenario: violates
- **Given** <a non-compliant situation — the fixture the projected test asserts the enforcer flags>
- **When** the enforcer runs
- **Then** it fails, pointing the author at the fix.

## Rationale log
<!-- Accretes; never rewrite history. -->
- **(<date>):** <why the rule is shaped this way>.
- **(<date>):** <each REVISE — a loosening / carve-out / tightening — appends a dated,
  justified entry here. This is the standard's amendment history (the ADR's living tail).>
```

---

## Notes

- **The `@spec` marker is `<standard>/<requirement>/<scenario>`.** A standard's scenarios are
  traceable units exactly like a functional capability's: mark each projected test with
  `@spec <standard-name>/<requirement>/<scenario>` — the standard name (the `standards/` filename
  stem) takes the capability slot. The conforms/violates scenarios are not decoration: each is
  the canonical, stack-agnostic statement of the property, and its test is a regenerable
  PROJECTION of it (re-project when a scenario changes). Coverage verifies, per touched/created
  standard, that EVERY `#### Scenario:` has ≥1 test carrying its marker — a scenario with no
  projecting test is declared-but-unprojected and FAILS. A single test MAY carry several markers,
  one per scenario it projects: a `mechanical` check covering BOTH conforms AND violates carries
  TWO (natural placement is on each test case / each valid|invalid fixture group). Scenario names
  match like requirement names (case-sensitive, trailing-whitespace normalized); renaming one ⇒
  re-project its marker.

- **The enforcer is named in the test, not its own field.** For a `mechanical` standard the
  enforcer is named in the projected test and the prose above — there is no `check:`,
  `invariant:`, or `binding:` field. Its FORM is the stack's choice: an executable test
  (unit / e2e / property / contract), a type-level check, a runtime assertion/contract, OR a
  lint/AST rule id (e.g. `eslint-plugin-local/require-rate-limit`). A behavioral scenario typically
  projects to an executable test; a purely structural rule MAY project to a lint/AST check — but
  lint is ONE option, not the default. Binding is DERIVED — a clause binds IFF its test resolves —
  so there is no `binding:` field to set.

- **Why the clause is a real `### Requirement:`.** The `### Requirement:` SHALL/MUST clause IS the
  authoritative one-liner; its conforms/violates scenarios are the PROJECTED FIXTURES the enforcer
  runs against. That is what makes "a test on every standard" concrete: the gate reads these
  scenarios for coverage and reads `governs:`+`tier:` for validation.

- **A self-contained record.** This file is plain git content under `standards/`, committed and
  read in place: the frontmatter, ADR, rule, scenarios, and rationale log all live HERE.
  (Functional capabilities are the ones that flow through `openspec/specs/` and archive natively —
  a different shape; see the `specs` artifact.)

- **Org-shared standards.** A standard is portable as-is: its rule + conforms/violates scenarios
  are stack-agnostic. To share across projects, copy the `standards/<name>.md` file into the
  org-standards bundle, pin it, and let the gate's integrity step assert the local copy matches the
  pin (you share the SCENARIO, each project PROJECTS its own local test). See the README and
  `templates/gate.md`.

- **Authoring aid.** The `engineering:architecture` skill is a good way to think through the
  Decision Record, but reshape its output into this layout.
