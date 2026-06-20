# project.md — the spec-first project contract

This is the ONE stack-specific file. It carries the four things the schema templates reference
abstractly. Copy it to `openspec/project.md` and fill the `<…>` slots.

> **Principle #0 — the spec is the source of truth.** This file wires the EXTERNAL checks that
> keep every derivative (code, test, check, migration) consistent with its spec. OpenSpec
> enforces none of it natively; the teeth are here.

---

```
## Gate

<Paste the `## Gate` + `## Traceability marker` block from templates/gate.md here and fill the
`<…>` command slots — the suite + org-integrity + coverage (per touched spec, EVERY `#### Scenario:`
has ≥1 test carrying its `@spec <capability>/<requirement>/<scenario>` marker — a scenario with no
projecting test FAILS; gate-verifies `tier:`; re-apply sub-check after CREATE) + regression +
`validate --strict` + judge pipeline, run continuously during a change and green as the last task
group.>

## Org-standards pin

- **Bundle:** <org-standards bundle name, or `none`>
- **Version:** <the pinned version the gate's integrity step matches `specs/_org/` against>

(Omit if you consume no org bundle. To change a pinned org rule, upstream a REVISE and bump this
version — never edit the local copy. See the README "Org-shared standards" section.)

## Branch protection (set at the git host — the gate canNOT enforce this)

The in-repo governance checks (changes-via-openspec, branch-per-change, change-lifecycle) only
bite if the host blocks every other way in. On the default branch, require:
- no direct pushes;
- a pull request to merge;
- the gate's CI check green (a required check);
- ≥ 1 human approval — which RATIFIES the spec as the source of truth (intent + scenario
  strength for functional specs, and any standards CREATE/REVISE), since the gate proves
  consistency among projections but not the source's correctness.

## Adoption mode

- **Mode:** greenfield | brownfield
- **Strict scope:** <whole repo | changed paths only>   # brownfield: changed paths until strict-clean once
- **Legacy:** pre-existing specs tagged `adoption: legacy` are coverage-exempt until first
  modified (then the exemption drops). See SETUP Step 0.
```

---

## Notes

- **Why four sections, not one.** Earlier drafts called the gate "the ONE project-specific
  thing"; in practice the org-standards pin, the branch-protection settings, and the adoption
  mode are equally project-specific and equally load-bearing. They live here so a reviewer can
  see the whole contract a project agreed to in one place.
- **The gate is embedded, not duplicated.** `templates/gate.md` is the authoritative gate block;
  this file just hosts it next to the other three sections. Keep them in sync by pasting, not
  paraphrasing.
- **Branch protection is the half OpenSpec and the repo cannot do.** Without it, "all changes via
  the flow" is advisory; with it, binding. Record the exact host settings so the guarantee is
  auditable.
- **The traceable unit is the SCENARIO.** Coverage keys on per-scenario `@spec` markers: each
  `#### Scenario:` is the canonical, stack-agnostic statement of a property, and its test is a
  LOCAL, regenerable PROJECTION of it (re-project when the scenario changes). The marker default
  is `@spec <capability>/<requirement>/<scenario>`, placed on each test CASE; one test MAY carry
  several markers (a conforms-AND-violates check carries two). The projection's FORM is the
  stack's choice — unit / e2e / property / contract / type-level / runtime assertion / lint-AST
  rule, whatever checks THAT scenario — so org-shared shares the SCENARIO, not a particular
  enforcer.
