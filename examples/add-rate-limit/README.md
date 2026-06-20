# Worked example — `add-rate-limit`

One change, walked end to end through the spec-first flow, against real files. The premise:
**add per-client rate limiting to the public HTTP API.** It produces TWO specs —

- a **functional capability**, `rate-limiter` (the limiter's behavior), and
- a **standard CREATE**, `rate-limit-policy` (`governs:` public HTTP handlers), caught on
  *ungoverned ground* by the design standards loop.

**Principle #0 in one change.** The specs under `change/specs/` are the **source of truth**.
Everything else here — the design, the tasks, the codemod, the AST check, the tests, the built
specs — is a **projection** of them. On divergence the spec is authoritative and changes first.
(The standard is cross-cutting across *this* project's whole public-router surface; sharing it
across other projects is not part of this example.)

> These files are **drafts to copy-adapt**, not a runnable repo. `change/` and `built/` are
> illustrative stand-ins for the real OpenSpec locations (`changes/add-rate-limit/` and
> `openspec/specs/`) so you can see the **before-archive** and **after-archive** states side by
> side without a live run.

## Read in this order

| File | What it is | Top-README section |
| --- | --- | --- |
| [`change/proposal.md`](change/proposal.md) | Why + What + **Affected surfaces & rollback** (feeds retrieval) | The flow & lifecycle |
| [`change/design.md`](change/design.md) | the **standards loop**: RETRIEVE finds nothing governs rate limiting → **CREATE** `rate-limit-policy` (tier `mechanical`); the limiter is functional, not a standard. design DECIDES; specs AUTHORS. | The standards loop |
| [`change/specs/rate-limiter/spec.md`](change/specs/rate-limiter/spec.md) | the **functional** delta — no `governs:`; scenarios become its tests | The model — two shapes |
| [`change/specs/rate-limit-policy/spec.md`](change/specs/rate-limit-policy/spec.md) | the **standard** delta — `governs:` present; conforms/violates scenarios double as the check's fixtures | The model — two shapes |
| [`change/tasks.md`](change/tasks.md) | resolve-then-author, `@spec` markers, the codemod migration, the continuous gate, archive, **post-archive re-apply** | Tests / coverage / gate |
| [`checks/rate-limit-policy.check.md`](checks/rate-limit-policy.check.md) | the `require-rate-limit` AST rule (the enforcer, named in prose — not a frontmatter field) + the RuleTester check-test (the standard's `test:`) | The gate |
| [`built/rate-limiter/spec.md`](built/rate-limiter/spec.md) | what `openspec/specs/rate-limiter/spec.md` looks like **after first archive + re-apply** | Empirical 1.4.1 facts |
| [`built/rate-limit-policy/spec.md`](built/rate-limit-policy/spec.md) | the built standard after re-apply — CORE + `## Decision Record` above `## Requirements` + trailing `## Rationale log` | Empirical 1.4.1 facts |
| [`project.md`](project.md) | the four-section contract: Gate, Org-pin (none), Branch protection, Adoption (greenfield) | The project.md contract |
| [`ci.yml`](ci.yml) | required gate job (steps 1–5) vs a separate non-required **soft** judge job | The gate / governance |

## Two shapes, one mechanism

The ONLY structural difference between the two specs is the `governs:` field. `rate-limiter`
omits it (functional — a component's behavior; its Given/When/Then scenarios become its tests).
`rate-limit-policy` carries it (a standard — cross-cutting; `governs:` the public router). Note
the loop in `design.md` only **decides** the standard; the spec is **authored** in `specs/` —
writing it in design would invert #0 (the recorded decision preceding its source spec).

## The gate goes RED, then GREEN

Run after each task group (`tasks.md` group 4). **Before the last group it is RED** for two
concrete reasons:

1. `rate-limit-policy` declares a `test:` that does not resolve yet (the check-test isn't wired)
   → **coverage** (gate step 3) fails.
2. The new `require-rate-limit` AST rule, once landed, **flags the 3 existing unwrapped public
   handlers** → **project mechanical checks** (gate step 1, `eslint .`) fail.

It goes **GREEN** only after: the limiter + its two scenario tests land (covers `rate-limiter`);
the AST rule + its check-test land with the `@spec` marker (covers `rate-limit-policy`); and the
**codemod** wraps the 3 handlers (clears the step-1 violations). The other steps: step 2
(org-integrity) is **skipped** (no bundle pinned); step 4 (regression) finds **no baseline** —
both specs are new, so coverage governs instead; step 5 (`validate --strict`) passes; step 6
(judge) runs but this change has **no judge-tier clause**, so it finds nothing (an empty judge
report is a valid, good outcome).

## The payoff: archive-on-CREATE + re-apply

This is the non-obvious moment most schemas only describe. On each new spec's **first archive
(CREATE)**, OpenSpec 1.4.1 rebuilds the main spec from the `## ADDED Requirements` delta **alone**
— it **drops** the frontmatter CORE, and for the standard the `## Decision Record` and
`## Rationale log`, and stubs `## Purpose` as `"TBD"`. (It also carries the delta body verbatim,
which is why the standard delta is kept **minimal** — no `#`-commented ADR is parked inside
`## ADDED Requirements`, or it would survive as cruft in the built `## Requirements`.)

So `tasks.md` group 6 **re-applies** the dropped content to `openspec/specs/<cap>/spec.md` and
re-fills Purpose — the `built/` files are the re-applied targets — then **re-runs the gate**.
Without the re-applied CORE the gate's coverage has no `test:` to read; and that re-apply is
**backstopped by a gate sub-check** — a built spec missing its CORE (`test:`) fails coverage,
so a forgotten re-apply cannot silently pass.

## Ratification — and what the gate does not prove

Because `rate-limit-policy` is a standard **CREATE**, it is surfaced for human sign-off at PR
review: the soft-CI judge posts a `"standards-changed"` suspect and a human approves; the change
is not self-merged. A green gate proves the projections are **consistent** with the specs, that no
scenario silently regressed, and that both specs are **enforced** — it does **not** prove the
specs are correct or their tests strong. That is what the human **ratifies** at PR review: green
proved consistency; the approval ratifies the spec as the source of truth.
