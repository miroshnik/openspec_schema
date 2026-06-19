# spec-first schema — setup

The correct way to put our flow into OpenSpec: a forked schema whose templates
carry the logic. This SUPERSEDES the old approach of stuffing the loop into
`config.rules` — the logic now lives in the flow itself, version-controlled, and
the generated skills come from your schema.

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

- Append the blocks in `templates/_additions.md` to the matching forked templates
  (`proposal.md`, `specs.md`, `design.md`, `tasks.md`). Keep the base content; add
  ours on top.
- Drop `templates/conformance.md` into the schema's `templates/`.
- Add ONLY the `conformance` artifact row to the forked `schema.yaml` — don't
  overwrite it. The fork already carries the correct base rows (with `template:`
  pointers), the `version:`/`description:` headers, and the `apply:` block. The
  `schema.yaml` here shows the full shape for reference.

## 3. Validate

```
openspec schema validate spec-first
```

## 4. Select it + regenerate skills

`openspec/config.yaml`:

```
schema: spec-first
context: |
  <project orientation / stack>     # ← the per-project slot
```

Then regenerate the tool artifacts from your schema:

```
openspec update
```

(Per-change override instead of project default:
`openspec new change <name> --schema spec-first`.)

## Where things live — the layering

- **The schema (templates) = the universal workflow.** Portable across every
  project. For cross-project reuse, copy the schema dir to
  `~/.local/share/openspec/schemas/` (user-global) — then it's available
  everywhere, and you maintain ONE copy of the flow.
- **`config.yaml` shrinks** to `schema:` + `context:`. The heavy logic now lives in
  templates, not in `config.rules`. (`config.rules` still works and still injects
  into templates — you just no longer need it for the loop.)
- **`project.md` keeps the ONE project-specific thing:** the gate definition
  (command, checks, traceability). The `tasks` template references "the project's
  gate" abstractly, so the schema stays stack-agnostic.

## On the `conformance` artifact (optional)

Modeling the judge as an artifact uses file-existence-as-state for something
re-runnable — the report is regenerated each run, not written once. It works, but
if that bothers you, omit `conformance` from `schema.yaml` and run the judge as a
hand-placed skill (`.claude/skills/`) or via the verify action. Either way it's a
SOFT pass: it surfaces suspects, a human triages, and the hard gate stays the
external mechanical checks + coverage + `openspec validate --strict`.
