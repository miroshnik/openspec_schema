# Worked example — `add-rate-limit`

One change, walked end to end through the spec-first flow, against real files. The premise:
**add per-client rate limiting to the public HTTP API.** It produces TWO source artifacts of
DIFFERENT shapes —

- a **functional capability**, `rate-limiter` (the limiter's behavior) — a PURE native OpenSpec
  spec, and
- a **standard CREATE**, `rate-limit-policy` (`governs:` public HTTP handlers) — a self-contained
  living record at `standards/rate-limit-policy.md`, caught on *ungoverned ground* by the design
  standards loop.

**The point.** Durable spec-first content lives OUTSIDE the change folder and OUTSIDE
`openspec/specs/`, so `openspec archive` never strips it. The functional spec archives NATIVELY
(its requirements survive on their own); the standard is a top-level file that archive never
touches. OpenSpec specs carry no custom frontmatter.

**Principle #0 in one change.** The `rate-limiter` spec under `change/specs/` and the
`rate-limit-policy` record under `standards/` are the **source of truth**. Everything else here —
the design, the tasks, the codemod, the AST check, the tests, the built spec — is a **projection**
of them. On divergence the source is authoritative and changes first. (The standard is cross-cutting
across *this* project's whole public-router surface; sharing it across other projects is a
secondary concern — a copy-in `standards/` bundle, pinned with an integrity check.)

> These files are **drafts to copy-adapt**, not a runnable repo. `change/` and `built/` are
> illustrative stand-ins for the real OpenSpec locations (`changes/add-rate-limit/` and
> `openspec/specs/`); `standards/` stands in for the real top-level `standards/` directory. You
> can see the **before-archive** and **after-archive** states side by side without a live run.

## Read in this order

| File | What it is | Top-README section |
| --- | --- | --- |
| [`change/proposal.md`](change/proposal.md) | Why + What + **Affected surfaces & rollback** (feeds retrieval) | The flow & lifecycle |
| [`change/design.md`](change/design.md) | the **standards loop**: RETRIEVE (reads `standards/`) finds nothing governs rate limiting → **CREATE** `rate-limit-policy` (tier `mechanical`) because the rule governs a class of public handlers; the limiter is functional, not a standard. design DECIDES; specs/standards AUTHORS. | The standards loop |
| [`change/specs/rate-limiter/spec.md`](change/specs/rate-limiter/spec.md) | the **functional** delta — pure native OpenSpec, no frontmatter; scenarios become its tests | The model — two shapes |
| [`standards/rate-limit-policy.md`](standards/rate-limit-policy.md) | the **standard** — a self-contained living record (`governs:`/`tier:` frontmatter; `## Decision Record`; `## Rule` with SHALL + conforms/violates; `## Rationale log`); never archived | The model — two shapes |
| [`change/tasks.md`](change/tasks.md) | ONE pass: author the spec + the standard record + tests + the codemod migration; LAST group = run the gate green; then archive + merge as the finish | Tests / coverage / gate |
| [`checks/rate-limit-policy.check.md`](checks/rate-limit-policy.check.md) | the `require-rate-limit` AST rule (the enforcer, named in the record's `## Rule` prose) + the RuleTester check-test (the standard's check) | The gate |
| [`built/rate-limiter/spec.md`](built/rate-limiter/spec.md) | what `openspec/specs/rate-limiter/spec.md` looks like **after archive** — a pure native spec, Purpose filled by OpenSpec's own step | Empirical 1.4.1 facts |
| [`project.md`](project.md) | the four-section contract: Gate, Org-pin (none), Branch protection, Adoption (greenfield) | The project.md contract |
| [`ci.yml`](ci.yml) | required gate job vs a separate non-required **soft** judge job | The gate / governance |

## Two shapes

The two source artifacts have DIFFERENT shapes and DIFFERENT homes:

- **`rate-limiter` — a functional capability.** A PURE native OpenSpec spec under `specs/`: a
  `## Purpose` + `## Requirements` with `### Requirement:` SHALL/MUST + `#### Scenario:`. No
  spec-first frontmatter at all. Its Given/When/Then scenarios become its tests. It survives
  archive natively.
- **`rate-limit-policy` — a standard.** A self-contained living record at
  `standards/rate-limit-policy.md` (top-level, NOT an OpenSpec spec, NEVER archived): `governs:` +
  `tier:` frontmatter, a `## Decision Record` (ADR), a `## Rule` carrying the `### Requirement:`
  SHALL clause with conforms/violates scenarios, and a `## Rationale log`. Everything about it —
  scope, tier, why, the rule, its scenarios, its amendment history — lives in that one file,
  always visible.

The loop in `design.md` only **decides** the standard; the record is **authored** at `standards/`
— writing the full decision in design would invert #0 (the recorded decision preceding its source).

## The gate goes RED, then GREEN

Run after each task group (`tasks.md` group 4), reading the CHANGE — the functional-spec delta,
the `standards/` record, the test markers, the full suite, and git, all of which exist pre-archive.
**Before the last group it is RED** for two concrete reasons:

1. `rate-limit-policy`'s check-test isn't wired yet, so its `conforms`/`violates` scenarios have no
   projecting test → **coverage** fails.
2. The new `require-rate-limit` AST rule, once landed, **flags the 3 existing unwrapped public
   handlers** → **project mechanical checks** (`eslint .`) fail.

It goes **GREEN** only after: the limiter + its two scenario tests land (covers `rate-limiter`);
the AST rule + its check-test land with the `@spec` markers (covers `rate-limit-policy`); and the
**codemod** wraps the 3 handlers (clears the eslint violations). The other steps: org-integrity is
**skipped** (no bundle pinned); standard-validation passes (the record has `governs`+`tier` + a
SHALL rule + conforms + violates); regression finds **no baseline** — both are new, so coverage
governs; `validate --strict` passes on the functional delta; the judge runs but this change has
**no judge-tier clause**, so it finds nothing (an empty judge report is a valid, good outcome).

## The finish: archive natively

Once the gate is green, archive + merge are the **finish**, not a task group with work after them:

- `openspec archive add-rate-limit` rebuilds `openspec/specs/rate-limiter/spec.md` from the
  `## ADDED Requirements` delta — the requirements survive natively. OpenSpec's OWN native step
  fills the stubbed `## Purpose` ("TBD … Update Purpose after archive"); the `built/` file shows it
  filled. That Purpose fill is OpenSpec's native chore.
- `standards/rate-limit-policy.md` is a plain git file — committed with the change, **untouched by
  archive**. Its frontmatter, Decision Record, and Rationale log are safe precisely because the
  file is never rebuilt. RETRIEVE reads `standards/` directly; the gate reads its scenarios for
  coverage and validates its shape.

The durable content never enters the archive path: the functional spec archives natively and the
standard is a committed git file archive leaves alone.

## Ratification — and what the gate does not prove

Because `rate-limit-policy` is a standard **CREATE**, it is surfaced for human sign-off at PR
review: the soft-CI judge posts a `"standards-changed"` suspect and a human approves; the change is
not self-merged. A green gate proves the projections are **consistent** with the source, that no
scenario silently regressed, and that both the spec and the standard are **enforced** — it does
**not** prove they are correct or their tests strong. That is what the human **ratifies** at PR
review: green proved consistency; the approval ratifies the spec/standard as the source of truth.
