# project.md — the spec-first project contract

This is the ONE stack-specific file. It carries the four things the schema templates reference
abstractly. Copy it to `openspec/project.md` and fill the `<…>` slots.

> **Principle #0 — the spec/standard is the source of truth.** This file wires the EXTERNAL
> checks that keep every derivative (code, test, check, migration) consistent with its spec or
> standard. OpenSpec enforces none of it natively; the teeth are here. Functional specs live in
> `openspec/specs/`; standards live in top-level `standards/` (outside `openspec/`, never
> archived) — code/tests are projections of both.

---

```
## Gate

<Paste the `## Gate` + `## Traceability marker` block from templates/gate.md here and fill the
`<…>` command slots — the MECHANICAL gate (the required CI status check) is: the full suite +
org-integrity (`standards/_org` bundle, normalized match) + coverage (per touched functional spec's
delta scenarios AND per touched/created `standards/<name>.md`, EVERY `#### Scenario:` has ≥1 test
carrying its `@spec <capability-or-standard>/<requirement>/<scenario>` marker — a scenario with no
projecting test FAILS; gate-verifies a standard's `tier:`; validates `standards/` structure) +
no silent scenario drop (preservation, vs git merge-base) + `validate --strict` (functional specs). NO LLM in CI. The judge is
a SEPARATE LOCAL step in the authoring loop (hard-local), not a CI gate step. Runs on the CHANGE as
the LAST TASK GROUP, green pre-archive; archive + merge are the FINISH after green.>

## Org-standards pin

- **Bundle:** <org-standards bundle name, or `none`>
- **Version:** <the pinned version the gate's integrity step matches `standards/_org/` against>

(Omit if you consume no org bundle. The org bundle is a COPY-IN under top-level `standards/_org/`;
the gate's integrity step matches it against this pin on normalized rule-bearing fields. To change
a pinned org rule, upstream a REVISE and bump this version — never edit the local copy. See the
README "Org-shared standards" section.)

## Branch protection (set at the git host — the gate canNOT enforce this)

The in-repo governance standards (changes-via-openspec, branch-per-change, change-lifecycle) only
bite if the host blocks every other way in. On the default branch, require:
- no direct pushes;
- a pull request to merge;
- the MECHANICAL gate's CI check green (the required status check) — suite + org-integrity +
  coverage + preservation + `validate --strict`. This is deterministic and runs no LLM; the judge is
  NOT a CI step (it runs locally in the authoring loop, hard-local);
- ≥ 1 human approval — which RATIFIES the source as the source of truth (intent + scenario
  strength for functional specs, and any standards CREATE/REVISE). The reviewer judges the
  un-mechanizable part (intent, naming, ergonomics, spirit) themselves and MAY run the judge on
  demand, since the gate proves consistency among projections but not the source's correctness. A
  PR that adds/edits a `standards/` record is flagged by a MECHANICAL diff check (git diff touches
  `standards/`) so the reviewer knows to ratify the standards change — no LLM needed to surface it.

## Adoption mode

- **Mode:** brownfield
- **Strict scope:** <whole repo | changed paths only>   # changed paths until strict-clean once
- **Coverage is CHANGE-SCOPED:** untouched functional specs and untouched `standards/` files are
  not audited; touch one → spec/standard it AND project its scenarios into tests. A non-blocking
  `<coverage --all>` backlog report drives the ratchet — brownfield is handled entirely by
  coverage being change-scoped. See SETUP Step 0.
```

---

## Notes

- **Why four sections.** The gate, the org-standards pin, the branch-protection settings, and the
  adoption mode are all project-specific and equally load-bearing. They live here so a reviewer can
  see the whole contract a project agreed to in one place.
- **The gate is embedded, not duplicated.** `templates/gate.md` is the authoritative gate block;
  this file just hosts it next to the other three sections. Keep them in sync by pasting, not
  paraphrasing.
- **Branch protection is the half OpenSpec and the repo cannot do.** Without it, "all changes via
  the flow" is advisory; with it, binding. Record the exact host settings so the guarantee is
  auditable.
- **Coverage is change-scoped, and that IS the brownfield story.** The gate audits only the
  functional-spec deltas and the `standards/` files this change touches or creates — a spec you
  never touch is never blocked, but touch it and you owe it a spec/standard plus per-scenario
  tests. The `<coverage --all>` report is the non-blocking backlog that ratchets the repo toward
  full coverage over time.
- **The traceable unit is the SCENARIO.** Coverage keys on per-scenario `@spec` markers across
  BOTH functional specs and `standards/` files: each `#### Scenario:` is the canonical,
  stack-agnostic statement of a property, and its test is a LOCAL, regenerable PROJECTION of it
  (re-project when the scenario changes). The marker default is
  `@spec <capability-or-standard>/<requirement>/<scenario>`, placed on each test CASE; one test
  MAY carry several markers (a conforms-AND-violates check carries two). The projection's FORM is
  the stack's choice — unit / e2e / property / contract / type-level / runtime assertion / lint-AST
  rule, whatever checks THAT scenario — so org-shared shares the SCENARIO, not a particular
  enforcer.
