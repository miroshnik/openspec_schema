# spec-first — an OpenSpec schema

A **spec-as-source-of-truth** workflow for [OpenSpec](https://github.com/Fission-AI/OpenSpec):
functional capabilities *and* cross-cutting standards live as specs, a standards loop
runs at design time, enforcement is tasked at the right tier, a single **gate** must be
green before a change completes, and an optional soft **LLM judge** surfaces conformance
suspects for a human (it never hard-blocks). Distributed as a community schema — you copy
it into your project.

## What's inside

```
openspec/schemas/spec-first/
├── schema.yaml              # full schema shape: 4 base artifacts + optional `conformance` + apply
├── SETUP.md                 # step-by-step install
└── templates/
    ├── _additions.md        # blocks to layer onto a spec-driven fork
    └── conformance.md       # the soft-judge template (the one complete new template)
```

## Why this is an OVERLAY, not a from-scratch schema

`spec-first` builds on a **fork of the built-in `spec-driven` schema**, so it inherits the
delta-spec format (`ADDED` / `MODIFIED` / `REMOVED` requirements) that `openspec archive`
and `openspec sync` rely on. The templates here are **additions** to that fork, not
standalone replacements — that's deliberate, and it's why install starts with a fork.

## Install

1. In your project: `openspec schema fork spec-driven spec-first`
2. Append the blocks in `templates/_additions.md` to the matching forked templates
   (`proposal` / `specs` / `design` / `tasks`). Keep the base content; add ours on top.
3. Copy `templates/conformance.md` into `openspec/schemas/spec-first/templates/`, and add
   the `conformance` row to your forked `schema.yaml` (see `schema.yaml` here — keep the
   fork's other rows; they carry the right `template:` pointers and the `apply:` block).
4. `openspec schema validate spec-first`
5. `config.yaml`: set `schema: spec-first` + `context: <your stack>`, then `openspec update`.

Full detail in `openspec/schemas/spec-first/SETUP.md`.

## Cross-project / team use

- **All your own projects:** drop the `spec-first/` folder in
  `~/.local/share/openspec/schemas/` (user-global) — it's then available everywhere, one
  copy of the flow.
- **A team:** this repo *is* the distribution. Teammates clone/copy the folder in. OpenSpec
  has **no remote-URL schema referencing** (you can't point a project at a GitHub schema by
  URL — see [issue #1131](https://github.com/Fission-AI/OpenSpec/issues/1131)); the bundle
  is brought local, the same way every community schema works.

## The project-specific knob

The `tasks` template references "the project's **gate**" abstractly. Each consuming project
defines what its gate actually runs (command, checks, traceability marker) in its own
`openspec/project.md` — so this schema stays stack-agnostic.
