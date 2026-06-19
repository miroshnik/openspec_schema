# Template additions

Append each block below to the matching template inside
`openspec/schemas/spec-first/templates/` AFTER forking. Keep the forked base
content (it defines the proposal/spec/design/task structure + the delta-spec
format); these blocks layer our flow on top.

---

## → append to `proposal.md`

```
## Affected surfaces & rollback
- List the surfaces this change touches — design uses this to retrieve the
  standards whose `governs:` scope covers them.
- State the rollback path.
```

---

## → append to `specs.md`

```
## Capabilities & standards (spec-first)

Every spec under specs/ is a capability. Two shapes, ONE mechanism — distinguished
by a `governs:` field in the frontmatter.

FUNCTIONAL capability = a component's behavior.
  Body: typed interface (provides/requires) + behavioral scenarios (Given/When/Then)
  + local invariants.

STANDARD = non-functional capability = a cross-cutting rule. Add `governs:` (the
surfaces it applies to).
  Body: the rule + a rationale-log (accretes v1/v2 — the ADR lives HERE, not in a
  separate doc) + conforms/violates examples + the check that enforces it + its tier.

Shared machine-read CORE — keep these in frontmatter, fixed place, so the gate and
design-retrieval find them regardless of body shape:
  invariant:  <the checkable condition>
  check:      <ref to the check that enforces it>      # or `none` = advisory
  tier:       mechanical | judge | advisory
  governs:    <scope>                                  # standards only; absent on functional
  binding:    DERIVED, not declared — a clause is binding IFF a check enforces it.
```

---

## → append to `design.md`

```
## Standards loop (spec-first)

Before writing the design, and whenever a cross-cutting decision arises:

RETRIEVE — pull the standards whose `governs:` matches the surfaces this change
touches (from the proposal). Apply them. Retrieval is first-try efficiency; the
gate is the backstop for misses.

Then for each cross-cutting decision:
  COMPLY  — fits an existing standard → follow it.
  CREATE  — hits ungoverned ground → propose a NEW standard (a spec with governs:,
            an invariant, a check, a tier). Bias to FEW — a standard taxes every
            future change. Creating one edits specs/ (fluid — expected, not a phase
            violation).
  REVISE  — in tension with an existing standard → write a delta to that standard's
            spec. Distinguish comply-vs-revise honestly; loosening or carve-outs
            need strong justification, logged in that standard's rationale-log (v2).

Surface every CREATE / REVISE for human ratification. Never silently invent law.
```

---

## → append to `tasks.md`

```
## Enforcement tasks (spec-first)

For every binding clause introduced or touched (a functional invariant or a standard):
  - Add/extend its CHECK at the correct TIER:
      mechanical (AST / lint / type / test) — the ONLY hard-binding tier; mechanize first.
      judge      — focused per-standard review of the un-mechanizable part
                   (intent / naming / ergonomics); SURFACES suspects, never hard-blocks.
      advisory   — prose only, no gate weight.
  - If the change breaks an existing standard's contract, add the MIGRATION — itself
    a projection of the change (codemod / data-migration / compat shim).

COVERAGE: a binding capability/standard with no resolvable check is
declared-but-unenforced — a defect, not a state. It must fail.

LAST GROUP = run the project's GATE (defined in project.md); it MUST be green before
the change is complete. The gate runs every mechanical check the project has + the
judge passes + the coverage check. New functional capabilities add their tests and a
traceability marker tying each test to its capability, so coverage sees they're covered.
```
