# spec-first schema — setup

The way to put our flow into OpenSpec: a forked schema whose artifact
`instruction:` fields carry the how-to logic, with clean fill-in `template:`
scaffolds. The logic lives in the flow itself, version-controlled, and reaches the
agent at authoring time (see step 4 for the exact mechanism).

> **Principle #0 — the spec/standard is the single source of truth.** Code, tests, checks,
> migrations, and docs are all projections of specs and standards; on divergence the spec is
> authoritative and changes first. The whole flow exists to enforce that.

> **The promise:** *an OpenSpec flow with checks you can trust your generation against, plus
> standards that are mandatory across the whole project (cross-cutting, enforced).* (The gate proves spec/test/code
> CONSISTENCY and no silent regression — not that the spec is true; that is what human
> ratification at PR review is for.)

> **Durable content lives OUTSIDE the change folder and OUTSIDE `openspec/specs/`.**
> A functional capability is a PURE native OpenSpec spec (a `## Purpose` + a `## Requirements`
> section) — its `## ADDED Requirements` delta rebuilds the main spec on archive. A standard is a
> self-contained living file at top-level `standards/<name>.md` — a committed git file, NOT an
> openspec spec, that archive does not touch. Archive merges the functional deltas into
> `openspec/specs/`; `standards/` files stay put.

## The two shapes

- **FUNCTIONAL CAPABILITY** = a pure native OpenSpec spec at `openspec/specs/<cap>/spec.md`: a
  `## Purpose` section + a `## Requirements` section (`### Requirement:` with SHALL/MUST +
  `#### Scenario:`). Its `## ADDED Requirements` delta rebuilds the main spec on archive; tests
  tie to scenarios via `@spec` markers. A brand-new spec is archived with OpenSpec's stubbed
  Purpose ("TBD … Update Purpose after archive"); you fill it.
- **STANDARD** = a self-contained living file at top-level `standards/<name>.md` (a sibling of
  `openspec/`, NOT an openspec spec, NEVER archived). Shape (see `templates/standard.md`):
  ```
  ---
  governs: <scope>
  tier:    mechanical | judge
  ---
  # <Standard name>
  ## Decision Record      (ADR: Status / Context / Decision / Consequences / Alternatives considered)
  ## Rule
  ### Requirement: <name> → The system SHALL <normative clause; literal SHALL/MUST>.
  #### Scenario: conforms …
  #### Scenario: violates …
  ## Rationale log → (<date>): …
  ```
  Everything about the standard — scope, tier, why, the rule, its scenarios, amendment history —
  lives HERE, always visible. RETRIEVE reads `standards/` directly; the gate reads its scenarios
  for coverage. The file is a committed git file that archive does not touch. Standards are NOT
  validated by `openspec validate` (they live outside `openspec/`) — the GATE validates them:
  `governs:` + `tier:` present, and a rule with SHALL/MUST + a `conforms` scenario + a
  `violates` scenario.

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

- Set the guidance blocks as the artifact `instruction:` fields in your forked `schema.yaml`
  (the proposal guidance on `proposal`, the functional-capability guidance on `specs`, the
  "Standards loop" design guidance on `design`, the standard-authoring guidance on the new
  `standard` artifact, and the "Enforcement tasks / coverage / regression" tasks guidance on
  `tasks`). The guidance is an `instruction:`, NOT appended to the template file. OpenSpec
  (verified 1.4.1) emits, per artifact, a clean `<template>` block ("Use this as the structure
  for your output file. Fill in the sections") AND a separate `<instruction>` block ("AI
  instructions for creating this artifact") — the how-to prose belongs in the latter. The
  `template:` files stay clean FILL-IN scaffolds; only genuine document STRUCTURE rides a
  template — add the small `## Affected surfaces & rollback` heading to the proposal template.
- Drop `templates/standard.md`, `templates/judge.md`, `templates/gate.md`,
  `templates/governance.md`, and `templates/project.md` into the schema's `templates/`.
  `standard.md` is the complete living-record shape authors copy when writing a STANDARD to
  `standards/<name>.md` (with its ADR + rationale log); `gate.md` is the gate block; `project.md`
  is the full project contract (it embeds the gate); `governance.md` seeds the process standards
  (all-via-flow, branch-per-change, lifecycle) as `standards/` records; `judge.md` is the judge.
- Add the `judge` AND the `standard` artifact rows to the forked `schema.yaml` — don't overwrite
  the base rows. The fork already carries the correct base rows (with `template:` pointers), the
  `version:`/`description:` headers, and the `apply:` block. Set the `instruction:` fields above
  on those base rows, and update `tasks` to `requires: [specs, design, standard]`. The new
  `standard` artifact `generates: ../../../standards/*.md` — TOP-LEVEL `<repo>/standards/`, OUTSIDE
  `openspec/`, so archive never touches it — and `requires: [design]`. The `schema.yaml` here shows
  the full shape for reference. (`standard.md` and `gate.md` are reference templates; only
  `standard` and `judge` are artifacts with schema.yaml rows.)

> **The standard artifact is CONDITIONAL.** It is authored/updated only when design's standards
> loop flags CREATE or REVISE; otherwise it is a no-op (no new `standards/` file). The DAG is
> `proposal → {specs, design}; design → standard; {specs, standard} → tasks → judge`.

> **Filename check.** Each artifact's `template:` must match a real file in `templates/`. The fork
> emits `spec.md` (the illustrative `schema.yaml` here uses `spec.md` to match); if your fork
> differs, reconcile the pointer to the on-disk name, or `openspec schema validate` fails.
> (Renaming the delta tokens inside a template — `## ADDED Requirements`, `### Requirement:`,
> `#### Scenario:`, `SHALL`/`MUST` — is a different, worse trap: it passes `schema validate` but
> silently breaks `validate`/`archive`. Don't.)

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
not baked into skills. The scaffold the agent fills in carries no how-to prose — guidance rides
the `<instruction>` block, the `template:` files stay clean fill-in scaffolds.

Smoke-test the install: `openspec new change probe`, then `openspec instructions specs --change
probe` — the functional-capability guidance must appear (now in the `<instruction>` block); the
`standard` artifact's instruction must surface the `standards/<name>.md` living-record shape.
If they don't, the prose isn't wired and the schema is inert. (Verified against OpenSpec 1.4.1.)

(Per-change override instead of project default:
`openspec new change <name> --schema spec-first`.)

## Where things live — the layering

- **The schema (`instruction:` + `template:`) = the universal workflow.** Portable across
  every project. For cross-project reuse, copy the schema dir to
  `~/.local/share/openspec/schemas/` (user-global) — then it's available
  everywhere, and you maintain ONE copy of the flow.
- **`openspec/specs/` holds functional capabilities** — pure native OpenSpec specs.
  **`standards/` (top-level, a sibling of `openspec/`) holds standards** — living records, plain
  committed git files. Two homes, two shapes; archive touches only the first.
- **`config.yaml` shrinks** to `schema:` + `context:`. The how-to logic now lives in the
  artifact `instruction:` fields, not in `config.rules`.
- **Additive per-team CONSTRAINTS go in `config.rules`** (per-artifact), distinct from the
  universal flow in `instruction:`/templates. e.g. "this team's proposals must always include
  a rollback plan" is a local constraint layered onto the `proposal` artifact via
  `config.rules` — it injects without forking the shared schema. The universal flow stays in
  `instruction:`; team-specific tightening rides `config.rules`.
- **`project.md` carries the project-specific CONTRACT** — three things: the gate definition,
  the org-standards pin, and the branch-protection settings. Copy `templates/project.md`. The
  schema templates stay stack-agnostic; this file is the one place a project meets the flow.
- **Org-shared standards (optional)** travel as a copy-in `standards/_org/` bundle, pinned in
  `project.md` and held immutable (integrity-checked) by the gate — see the README's "Org-shared
  standards" section.

## 5. Define the gate (required — this is what makes the flow bite)

Nothing in spec-first is enforced by `openspec` itself — `validate`/`archive` have no
coverage/test-per-spec/gate primitive, and standards live outside `openspec/` entirely. The teeth
are external. The gate is the deterministic, MECHANICAL gate — steps 1–6 below, no LLM. It is what
CI runs as the required status check (cheap, reproducible, needs no API keys) AND what you run
locally after each task group. Branch protection requires THIS check. Paste `templates/gate.md`
into `openspec/project.md` and wire a single command that runs, PRE-archive (everything it reads —
functional-spec deltas, `standards/`, tests, the full suite, git — exists before archive):

1. the full project check/test suite (regression guard — the WHOLE suite, not just this
   change's checks);
2. **org-standards integrity** (only if you consume an org bundle) — every `standards/_org/`
   record matches the pinned bundle on its normalized rule-bearing fields, so a locally loosened
   org rule fails;
3. the **standards validation** — each touched/created `standards/<name>.md` is well-formed:
   `governs:` + `tier:` present, a rule with literal SHALL/MUST, a `conforms` scenario, and a
   `violates` scenario. (Standards are not seen by `openspec validate`; the gate is their
   validator.);
4. the **coverage** check — per FUNCTIONAL spec touched by the change (its delta) AND per
   touched/created STANDARD, every `#### Scenario:` has ≥1 test carrying its
   `@spec <capability-or-standard>/<requirement>/<scenario>` marker; a scenario with no projecting
   test is declared-but-unprojected and FAILS. It reads the CHANGE (functional-spec deltas) + the
   `standards/` files + test markers + git. (Judge clauses covered by their registered judge run.
   Coverage is CHANGE-SCOPED: untouched legacy specs are not audited.);
5. the **regression diff** — each touched functional spec's requirement+scenario set vs its
   merge-base built spec (from git), and each touched `standards/` file diffed in git, catching
   silently dropped scenarios;
6. `openspec validate --strict` (functional specs only) — tested on OpenSpec 1.4.1: `## Purpose`
   < 50 chars fails ONLY under `--strict`; the structural rules (Purpose + Requirements present,
   each requirement a literal SHALL/MUST + ≥1 Scenario) are HARD errors that fail even without it.
   (`## Why` < 50 and > 10 deltas do NOT fail strict in 1.4.1 — add those as your own grep if you
   want them.);
Steps 1–6 ARE the mechanical gate. The **judge** for tier-`judge` clauses is a SEPARATE LOCAL step
in the authoring loop — a local in-loop reviewer (hard-local): the loop runs it and won't close a
task with an open suspect (resolve it, or mark it justified with a pointer to the standard's
Rationale log). It is NOT a CI job and NOT a step in the gate CI runs; CI stays mechanical-only and
the human ratifies the un-mechanizable part at PR review.

Run the mechanical gate after every task group during a change, and require it green as the LAST
task group —
everything it reads exists pre-archive. Implement the coverage + regression checks once per
project; they key off the traceability marker (`@spec <capability-or-standard>/<requirement>/<scenario>`
by default — one per scenario a test projects; a check covering conforms AND violates carries two).
Each test is a stack-specific PROJECTION of the scenario — its FORM is the stack's choice (unit /
e2e / property / contract / type-level / runtime assertion / lint-AST rule — lint is ONE option,
never the default; tier is machine-checkable-at-all vs needs-human-judgment, NOT the mechanism).
Org-shared shares the SCENARIO; each consuming project projects its own test.

## 6. The flow — one pass, then archive as the finish

The user model:

```
proposal → specs → design → (standard, when a cross-cutting decision arises) → tasks
```

`tasks` is EVERYTHING in ONE pass: author the functional specs + the `standards/` records + the
tests + the checks + the migration. The LAST TASK GROUP runs the GATE green — the gate reads the
CHANGE (functional deltas + `standards/` + tests + the full suite + git), all of which exist
pre-archive.

THEN **archive + merge as the finish**: `openspec archive` merges the functional-spec deltas into
`openspec/specs/` and moves the change to `changes/archive/`; the `standards/` files are committed
git files, untouched by archive. Merge is the finish. (A brand-new functional spec is archived with
OpenSpec's stubbed `## Purpose` "TBD … Update Purpose after archive"; you fill it.)

## 7. Brownfield — change-scoped coverage + the ratchet

spec-first governs **changes**, not your pre-existing tree, so it installs green on a real repo and
ratchets coverage upward one change at a time. Brownfield is handled structurally:

- **Coverage is CHANGE-SCOPED.** It audits only specs the change touches and standards it
  touches/creates; untouched legacy specs are not audited. Touch a spec → you spec-and-test it
  ("you touched it, you specify it"). This is the ratchet: each change fully specs and tests what
  it adds or modifies, and the covered surface grows monotonically.
- **Scope strict to changed paths** (or bring the repo `validate --strict`-clean once) so an
  unrelated change never reds on legacy well-formedness.
- A non-blocking `coverage --all` backlog report shows the still-unspecified surface without
  blocking the change — the ratchet's dashboard, not a gate.

(Greenfield: nothing to ratchet; coverage simply applies to every change from the start.)

## 8. Bootstrap standards from existing conventions (brownfield on-ramp)

On a fresh adoption the standards loop's RETRIEVE step finds nothing in `standards/`, so every
cross-cutting decision looks like ungoverned CREATE ground. Seed the corpus from what you ALREADY
enforce:

- For each existing rule (eslint/biome config, tsconfig flags, CI checks, CONTRIBUTING
  conventions), write a `standards/<name>.md` record whose test RESOLVES to that existing tool —
  name the enforcer (e.g. the eslint/AST rule id like `eslint-plugin-local/require-rate-limit`) in
  the rule prose and the test body. This is resolve-then-author documenting enforcement you already
  have, not new machinery.
- Give each a `governs:` scope and a `conforms`/`violates` scenario pair, and bias to FEW.
- This gives design-time RETRIEVE something real to find in `standards/`, and turns implicit
  conventions into enforced, reviewable standards one at a time.

The loop's per-decision branches are COMPLY / CREATE / REVISE / DEFER (see `_additions.md`).
DEFER keeps a decision LOCAL — it stays the functional capability's behavior (its scenarios) and is
RECORDED as a standard CANDIDATE — when bias-to-FEW says it is too early to standardize (one
surface, not clearly a class yet); promote it via CREATE + a migration once a second surface needs
the same rule or a reviewer flags it. A standard is a rule that has OUTGROWN one capability; you do
not standardize on a hunch.

## 9. Governance — make the flow the only path (recommended)

To enforce "every change goes through OpenSpec," adopt the seed standards in
`templates/governance.md` as `standards/` records (or `standards/_org/` to make them org-wide and
immutable). They are ordinary standards — their checks run in the gate's suite (step 1):

- **changes-via-openspec** — the default branch accepts only archived OpenSpec changes.
- **branch-per-change** — one change per branch/worktree, named `change/<id>`.
- **change-lifecycle** — gate-green + approved + archived before merge.

The repo half is those checks. The other half lives at your git host: **protect the default
branch** — no direct pushes, require a PR, require the gate's CI check green, require ≥1
approval — and record those settings in `project.md`. Without branch protection "all via flow"
is advisory; with it, binding. The full START → AUTHOR → BUILD → GATE → ARCHIVE → PR → REVIEW →
MERGE lifecycle is in `governance.md`.

## On the `judge` artifact (optional)

Modeling the judge as an artifact uses file-existence-as-state for something
re-runnable — the report is regenerated each run, not written once. It works, but
if that bothers you, omit `judge` from `schema.yaml` and run the judge as a
hand-placed skill (`.claude/skills/`) or via the verify action. Either way its posture
is the same: a LOCAL in-loop reviewer (hard-local) — the authoring loop won't close a
task with an unresolved suspect (resolve it, or mark it justified with a pointer to the
standard's Rationale log). It is NOT a CI job: CI runs only the deterministic, mechanical
gate (no LLM, no API keys, reproducible), and the human ratifies the un-mechanizable part
at PR review and MAY run the judge on demand. A team that wants LLM suspects posted on PRs
MAY add their own non-required judge-annotation job, but it is not shipped by default. The
required CI check stays the mechanical gate — the project suite + org-standards integrity +
standards validation + coverage + regression +
`openspec validate --strict` — a non-deterministic verdict never reds a shared build.
