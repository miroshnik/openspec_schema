# spec-first schema — setup

The correct way to put our flow into OpenSpec: a forked schema whose artifact
`instruction:` fields carry the how-to logic, with clean fill-in `template:`
scaffolds. This SUPERSEDES the old approach of stuffing the loop into
`config.rules` — the logic now lives in the flow itself, version-controlled, and
reaches the agent at authoring time (see step 4 for the exact mechanism).

> **Principle #0 — the spec is the single source of truth.** Code, tests, checks, migrations,
> and docs are all projections of specs; on divergence the spec is authoritative and changes
> first. The whole flow exists to enforce that.

> **The promise:** *an OpenSpec flow with checks you can trust your generation against, plus
> standards that are mandatory across the whole project (cross-cutting, enforced).* (The gate proves spec/test/code
> CONSISTENCY and no silent regression — not that the spec is true; that is what human
> ratification at PR review is for.)

## Step 0 — adoption mode (brownfield)

spec-first governs **changes**, not your pre-existing tree — so it installs green on a
real repo and ratchets coverage upward one change at a time. Before the first change:

- **Scope strict to changed paths** (or bring the repo `validate --strict`-clean once) so
  an unrelated change never reds on legacy well-formedness.
- **Tag every pre-existing spec `adoption: legacy`** in its frontmatter — its mere PRESENCE
  exempts the spec from the coverage check until it is next modified, when you drop the field
  ("you touched it, you specify it").
- Then **enforce-on-touch**: each change fully specs and tests what it adds or modifies.

(Greenfield: skip this — there's no legacy tree, so no spec carries `adoption:` at all; an
omitted `adoption` is an active spec, fully covered.)

## 1. Fork the stock schema

Fork — don't write from scratch. The fork inherits the delta-spec format
(`ADDED` / `MODIFIED` / `REMOVED` requirements) that `openspec archive` and
`openspec sync` depend on. A hand-written specs template that drops that format
breaks archiving.

```
openspec schema fork spec-driven spec-first
```

Writes `openspec/schemas/spec-first/` (project-local, version-controlled).

## 2. Layer in our flow

- Set the four guidance blocks as the artifact `instruction:` fields in your forked
  `schema.yaml` (the proposal guidance on `proposal`, the "Capabilities & standards"
  specs guidance on `specs`, the "Standards loop" design guidance on `design`, and the
  "Enforcement tasks / coverage / regression / re-apply" tasks guidance on `tasks`). The
  guidance is now an `instruction:`, NOT appended to the template file. OpenSpec (verified
  1.4.1) emits, per artifact, a clean `<template>` block ("Use this as the structure for your
  output file. Fill in the sections") AND a separate `<instruction>` block ("AI instructions
  for creating this artifact") — the how-to prose belongs in the latter. The `template:` files
  stay clean FILL-IN scaffolds; only genuine document STRUCTURE rides a template — add the
  small `## Affected surfaces & rollback` heading to the proposal template.
- Drop `templates/judge.md`, `templates/standard.md`, `templates/gate.md`,
  `templates/governance.md`, and `templates/project.md` into the schema's `templates/`.
  `standard.md` is the complete archive-safe shape authors copy when writing a STANDARD (with
  its ADR); `gate.md` is the gate block; `project.md` is the full project contract (it embeds
  the gate); `governance.md` seeds the process standards (all-via-flow, branch-per-change,
  lifecycle); `judge.md` is the judge.
- Add ONLY the `judge` artifact row to the forked `schema.yaml` — don't
  overwrite it. The fork already carries the correct base rows (with `template:`
  pointers), the `version:`/`description:` headers, and the `apply:` block. Set the
  `instruction:` fields above on those base rows. The `schema.yaml` here shows the full shape
  for reference. (`standard.md` and `gate.md` are reference templates, not artifacts — no
  schema.yaml row.)

> **Filename check.** Each artifact's `template:` must match a real file in `templates/`.
> The fork emits `spec.md` (the illustrative `schema.yaml` here uses `spec.md` to match); if
> your fork differs, reconcile the pointer to the on-disk name, or
> `openspec schema validate` fails. (Renaming the delta tokens inside a template —
> `## ADDED Requirements`, `### Requirement:`, `#### Scenario:`, `SHALL`/`MUST` — is a
> different, worse trap: it passes `schema validate` but silently breaks
> `validate`/`archive`. Don't.)

## 3. Validate

```
openspec schema validate spec-first
```

## 4. Select it + how the prose reaches the agent

`openspec/config.yaml`:

```
schema: spec-first
context: |
  <project orientation / stack>     # ← the per-project slot
```

Then:

```
openspec update
```

`openspec update` writes only the GENERIC skill/command wrappers from the installed CLI — they
carry NONE of your guidance prose. Your prose binds the agent at AUTHORING time: the wrappers
call `openspec instructions <artifact> --change <name>`, which emits both the forked
`<template>` (clean fill-in scaffold) and the `<instruction>` block (your guidance, set as the
artifact's `instruction:` field). So the logic lives in the schema and is delivered on demand,
not baked into skills. Because the guidance no longer rides in `<template>`, the old caveat
about copying guidance into `changes/<id>/specs/*.md` is gone — the hazard is dissolved at the
root, since the scaffold the agent fills in carries no how-to prose.

Smoke-test the install: `openspec new change probe`, then `openspec instructions specs --change
probe` — the "Capabilities & standards" guidance must appear (now in the `<instruction>` block).
If it doesn't, the prose isn't wired and the schema is inert. (Verified against OpenSpec 1.4.1.)

(Per-change override instead of project default:
`openspec new change <name> --schema spec-first`.)

## Where things live — the layering

- **The schema (`instruction:` + `template:`) = the universal workflow.** Portable across
  every project. For cross-project reuse, copy the schema dir to
  `~/.local/share/openspec/schemas/` (user-global) — then it's available
  everywhere, and you maintain ONE copy of the flow.
- **`config.yaml` shrinks** to `schema:` + `context:`. The how-to logic now lives in the
  artifact `instruction:` fields, not in `config.rules`.
- **Additive per-team CONSTRAINTS go in `config.rules`** (per-artifact), distinct from the
  universal flow in `instruction:`/templates. e.g. "this team's proposals must always include
  a rollback plan" is a local constraint layered onto the `proposal` artifact via
  `config.rules` — it injects without forking the shared schema. The universal flow stays in
  `instruction:`; team-specific tightening rides `config.rules`.
- **`project.md` carries the project-specific CONTRACT** — four things: the gate definition,
  the org-standards pin, the branch-protection settings, and the adoption mode. Copy
  `templates/project.md`. The schema templates stay stack-agnostic; this file is the one place
  a project meets the flow.
- **Org-shared standards (optional)** travel as a copy-in `org-standards/` bundle seeded
  into `specs/_org/`, pinned in `project.md` and held immutable by the gate — see the
  README's "Org-shared standards" section.

## 5. Define the gate (required — this is what makes the flow bite)

Nothing in spec-first is enforced by `openspec` itself — `validate`/`archive` have no
`governs:`/coverage/test-per-spec/gate primitive. The teeth are external. Paste
`templates/gate.md` into `openspec/project.md` and wire a single command that runs:

1. the full project check/test suite (regression guard — the WHOLE suite, not just this
   change's checks);
2. **org-standards integrity** (only if you consume an org bundle) — every `specs/_org/`
   spec matches the pinned bundle on its normalized rule-bearing fields, so a locally loosened
   org rule fails;
3. the **coverage** check — per spec TOUCHED BY THE CHANGE, every `#### Scenario:` has ≥1 test
   carrying its `@spec <capability>/<requirement>/<scenario>` marker; a scenario with no
   projecting test is declared-but-unprojected and FAILS (judge clauses covered by their
   registered judge run; `adoption: legacy` specs exempt until modified);
4. the **regression diff** — each touched main spec's requirement+scenario set vs its
   merge-base built spec (from git, NOT the archive delta), catching silently dropped scenarios;
5. `openspec validate --strict` — tested on OpenSpec 1.4.1: `## Purpose` < 50 chars fails ONLY
   under `--strict`; the structural rules (Purpose + Requirements present, each requirement a
   literal SHALL/MUST + ≥1 Scenario) are HARD errors that fail even without it. (`## Why` < 50
   and > 10 deltas do NOT fail strict in 1.4.1 — add those as your own grep if you want them.);
6. the **judge** for tier-`judge` clauses — blocking locally, soft (post suspects) in CI.

Run it after every task group during a change, and require it green as the last task group.
Implement the coverage + regression checks once per project; they key off the traceability
marker (`@spec <capability>/<requirement>/<scenario>` by default — one per test case; a check
covering conforms AND violates carries two). Each test is a stack-specific PROJECTION of the
scenario — its FORM is the stack's choice (unit / e2e / property / contract / type-level /
runtime assertion / lint-AST rule, whatever checks THAT scenario); org-shared shares the
SCENARIO, and each consuming project projects its own test.

## 6. Bootstrap standards from existing conventions (brownfield on-ramp)

On a fresh adoption the standards loop's RETRIEVE step finds nothing, so every cross-cutting
decision looks like ungoverned CREATE ground. Seed the corpus from what you ALREADY enforce:

- For each existing rule (eslint/biome config, tsconfig flags, CI checks, CONTRIBUTING
  conventions), write a STANDARD whose `test:` RESOLVES to that existing tool — name the
  enforcer (e.g. the eslint/AST rule id like `eslint-plugin-local/require-rate-limit`) in the
  test body and the spec prose, not in frontmatter. This is resolve-then-author documenting
  enforcement you already have, not new machinery.
- Tag each `adoption: legacy`, mark it a standard with `governs:` (its presence is what makes
  the spec a standard), and bias to FEW.
- This gives design-time RETRIEVE something real to find, and turns implicit conventions
  into enforced, reviewable standards one at a time.

## 7. Governance — make the flow the only path (recommended)

To enforce "every change goes through OpenSpec," adopt the seed standards in
`templates/governance.md` under `specs/governance/` (or `specs/_org/` to make them org-wide
and immutable). They are ordinary standards — their checks run in the gate's suite (step 1):

- **changes-via-openspec** — the default branch accepts only archived OpenSpec changes.
- **branch-per-change** — one change per branch/worktree, named `change/<id>`.
- **change-lifecycle** — gate-green + approved + archived before merge.

The repo half is those checks. The other half lives at your git host: **protect the default
branch** — no direct pushes, require a PR, require the gate's CI check green, require ≥1
approval — and record those settings in `project.md`. Without branch protection "all via flow"
is advisory; with it, binding. The full START → AUTHOR → BUILD → ARCHIVE → PR → REVIEW → MERGE
lifecycle is in `governance.md`.

## On the `judge` artifact (optional)

Modeling the judge as an artifact uses file-existence-as-state for something
re-runnable — the report is regenerated each run, not written once. It works, but
if that bothers you, omit `judge` from `schema.yaml` and run the judge as a
hand-placed skill (`.claude/skills/`) or via the verify action. Either way its posture
is the same: HARD locally (the authoring loop won't close a task with an unresolved
suspect), SOFT in CI (it posts suspects, a human triages). The CI hard gate stays the
external mechanical checks + coverage + regression + `openspec validate --strict` — a
non-deterministic verdict never reds a shared build.
