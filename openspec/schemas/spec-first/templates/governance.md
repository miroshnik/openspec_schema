# Governance — "all changes go through the flow" as living standards

The strongest rule a spec-first repo can set is **process** itself: nothing reaches the
default branch except through a reviewed, gated, archived change. spec-first has a home for
exactly this — a **standard**. So governance rules are not a new mechanism; they are ordinary
standards (the schema dogfooding itself), enforced by the same gate, plus one out-of-repo piece
(branch protection) that no in-repo check can replace.

These three are **standards**, not OpenSpec specs. Drop each as a SELF-CONTAINED record at
`standards/<name>.md` (top-level `<repo>/standards/`, OUTSIDE `openspec/` — so `openspec archive`
never touches it). They follow `templates/standard.md`: `governs:`+`tier:` frontmatter, a
`## Decision Record`, a `## Rule` (a `### Requirement:` with literal SHALL/MUST + conforms/violates
scenarios), and a `## Rationale log`. Everything about the rule lives in the file, always visible.
The enforcer named in each standard's `## Rule` test is stack-specific; wire it into the gate
(`templates/gate.md` step 1 — it runs in the project suite like any standard's enforcer). The gate
reads the scenarios for coverage; `openspec validate` never sees these files.

> **Honesty about enforcement.** "No direct pushes to the default branch" is enforced by your
> git host's **branch protection / required checks**, NOT by anything in the repo. The repo
> ships the *checks that run on a PR*; the host ships the *rule that a PR is the only way in*.
> Both are required — document the branch-protection settings alongside these standards.

---

## Seed standard 1 — `standards/changes-via-openspec.md` (the architectural one)

```
---
governs: the default branch — every commit that lands on it
tier:    mechanical            # structural checks are mechanical; "diff matches intent" is a judge backstop
---

# Changes via OpenSpec

## Decision Record
- **Status:** accepted
- **Context:** ad-hoc commits bypass specs, tests, and the gate — the whole trust guarantee leaks through that gap.
- **Decision:** the default branch accepts changes ONLY via an OpenSpec change merged through a PR; direct pushes are blocked at the host.
- **Consequences:** every change is specced, tested, gated, and archived; even a hotfix is one fast change. Trade-off: no out-of-band commits, ever.
- **Alternatives considered:** honor-system / docs-only → unenforced, drifts. CODEOWNERS → governs *who reviews*, not *flow-conformance*.

## Rule

### Requirement: Every merge to the default branch is an archived OpenSpec change
The default branch SHALL accept commits only through a PR that archives exactly one OpenSpec
change, and direct pushes to it MUST be rejected.

#### Scenario: conforms
- **Given** a PR from `change/add-auth` whose diff archives `add-auth`
- **When** the enforcer `ci/changes-via-openspec` runs — it asserts the diff adds a `changes/archive/<date>-<id>/` tree and every changed NON-GENERATED source path is referenced by that change's tasks.md or a functional spec (lockfiles/vendored/regenerated paths excluded)
- **Then** the check passes.

#### Scenario: violates
- **Given** a PR that edits `src/` with no OpenSpec change in the diff
- **When** `ci/changes-via-openspec` runs
- **Then** it fails: "no OpenSpec change — author one with `openspec new change`."

## Rationale log
- **<date>:** mandatory. Narrow exceptions (bot dependency bumps) still go through a `chore/<id>` change, never a bypass. The push-protection leg (no direct pushes to the default branch) is asserted OUT-OF-BAND by branch protection, NOT by this script — the in-repo half is the archive+reference structure; see the honesty note above.
```

---

## Seed standard 2 — `standards/branch-per-change.md`

```
---
governs: git branches / worktrees that hold OpenSpec changes
tier:    mechanical
---

# One branch per change

## Decision Record
- **Status:** accepted
- **Context:** a change is the unit of review and of isolation; mixing two on one branch makes PRs, gates, and archives ambiguous.
- **Decision:** each change lives on its own branch (or worktree) named `change/<change-id>`, id matching the branch, exactly one change per branch.
- **Consequences:** PRs, gates, and archives stay strictly one-to-one and reviewable. Trade-off: a stacked sequence of changes is a sequence of branches, not one branch.
- **Alternatives considered:** many changes per feature branch → archive/gate scope blurs. Trunk-based commits → no isolation unit at all.

## Rule

### Requirement: A branch introduces exactly one change, named for it
Each OpenSpec change SHALL live on its own branch named `change/<change-id>` (id matching the
branch name), and a branch MUST NOT introduce more than one change — whether that change is
still active or already archived in the diff.

#### Scenario: conforms
- **Given** branch `change/add-auth` introducing exactly one change — `changes/add-auth/` before archive, or `changes/archive/<date>-add-auth/` in the PR diff after archive
- **When** the enforcer `ci/branch-per-change` runs — it asserts exactly ONE change is introduced by this branch (an active `changes/<id>/` pre-archive, OR a newly-added `changes/archive/<date>-<id>/` in this PR's diff post-archive) AND its `<id>` == the branch name with the `change/` prefix stripped
- **Then** `branch-per-change` passes in both states.

#### Scenario: violates
- **Given** a branch introducing two changes, or `changes/add-auth/` on branch `feature/x`
- **When** `ci/branch-per-change` runs
- **Then** it fails, naming the mismatch.

## Rationale log
- **<date>:** worktrees count — `git worktree add ../add-auth -b change/add-auth`. The rule is name + singularity, not branch-vs-worktree.
```

---

## Seed standard 3 — `standards/change-lifecycle.md`

```
---
governs: the BUILD → PR → review → archive → merge transition of every change
tier:    mechanical            # the in-repo half (archive present in the merge diff) is mechanical; gate-green + approval are host-enforced
---

# Change lifecycle

## Decision Record
- **Status:** accepted
- **Context:** specs, gate, and archive can each be skipped on the way to the default branch unless an end state is pinned and checked.
- **Decision:** a change reaches the default branch ONLY when its gate is green, its PR is human-approved, and `openspec archive` has run — the FINISH order is BUILD (gate green) → PR → review → archive → merge.
- **Consequences:** the merged diff always carries the updated `openspec/specs/`, the gate always ran against everything (deltas + standards/ + tests), and a human ratified the source. Trade-off: a merge is never a one-keystroke fast-forward — it carries an archive commit.
- **Alternatives considered:** merge-then-archive → the default branch transiently holds un-archived specs. Gate-after-merge → the bad state already landed.

## Rule

### Requirement: Gate-green, approved, and archived before merge
A change SHALL reach the default branch only when its gate is green, its PR is human-approved,
and `openspec archive` has run so the merged diff carries the updated `openspec/specs/`. A merge
missing any of the three MUST be rejected. The approval RATIFIES THE SPEC AS THE SOURCE OF TRUTH —
a functional capability's intent and the strength of its scenarios, and any standards CREATE/REVISE —
because the gate proves consistency among projections but not the source's correctness.

#### Scenario: conforms
- **Given** a PR whose gate is green, that is approved, and whose diff archives the change (functional-spec deltas merged into `openspec/specs/`; the `standards/` files are plain committed git files, untouched by archive)
- **When** merge is attempted — the enforcer `ci/change-lifecycle` asserts the archive is present in the PR diff; gate-green and ≥1 approval are required-checks enforced out-of-band by branch protection
- **Then** it is allowed.

#### Scenario: violates
- **Given** a PR that is green and approved but did NOT run `openspec archive`
- **When** merge is attempted
- **Then** `ci/change-lifecycle` rejects it: "archive the change before merge."

## Rationale log
- **<date>:** the check verifies the END STATE (all three present at merge), not a strict temporal order, so PRs can open early; the LAST commit before merge is typically `openspec archive`. If your host dismisses approvals on new commits, re-request after archiving.
- **<note>:** only the archive-in-diff leg is mechanically checkable in-repo; gate-green and approval are host-enforced (required checks + branch protection) with no in-repo fixture — same split as the honesty note at the top of governance.md.
- **<note>:** archive moves only the FUNCTIONAL-spec deltas into `openspec/specs/`; `standards/` is top-level and never archived, so the gate has already validated those records pre-archive and merge carries them as ordinary committed files.
```

---

## The change lifecycle, step by step

```
1. START   — `openspec new change <id>` and branch: `git worktree add ../<id> -b change/<id>`
             (or `git switch -c change/<id>`). Convention: change id == branch suffix; ONE change per branch.
2. AUTHOR  — proposal → specs → design → (standard, when a cross-cutting decision arises) → tasks.
             design DECIDES standards; tasks AUTHORS them. Functional specs land at
             `openspec/specs/<cap>/spec.md`; standards land at `standards/<name>.md`.
3. BUILD   — tasks is ONE pass: author functional specs + `standards/` records + tests + checks +
             migration. Run the GATE after each task group; the LAST TASK GROUP is the gate GREEN.
             The gate reads the CHANGE itself — functional-spec deltas + the `standards/` files +
             test markers + the full suite + git — and everything exists PRE-archive.
4. PR      — open the PR from `change/<id>`. CI runs the mechanical gate (the required check; no
             LLM); the human reviewer ratifies at review. The changes-via-openspec,
             branch-per-change, and change-lifecycle checks must pass.
5. REVIEW  — a human approves, ratifying the spec as source of truth and any standards CREATE/REVISE.
             Branch protection requires approval + all required checks green.
6. ARCHIVE — `openspec archive <id>` on the branch → merges the FUNCTIONAL-spec deltas into
             `openspec/specs/` and moves the change to `changes/archive/`. `standards/` files are
             committed git files, untouched by archive. A brand-new functional spec is archived
             with OpenSpec's stubbed `## Purpose` ("TBD … Update Purpose after archive"); you fill
             it after archive.
7. MERGE   — squash-merge to the default branch. Direct pushes are blocked at the host, so this PR
             is the only way in — closing the loop with `changes-via-openspec`.
```

## Wiring (per project)

- **In repo (version-controlled):** the three enforcers named in the standards' `## Rule` tests above
  (`ci/changes-via-openspec`, `ci/branch-per-change`, `ci/change-lifecycle`), run by the gate (suite,
  step 1) and scenario-marked like any standard's enforcer (`@spec changes-via-openspec/<requirement>/conforms`,
  etc.). Implement once; they key off `changes/`, the branch name, and the PR diff. The standards/
  records themselves are plain git files — never an `openspec validate` target; the gate validates them.
- **At the git host (cannot live in the repo):** protect the default branch — no direct pushes,
  require a PR, require the gate's CI check green, require ≥1 approval. This is the half that makes
  "all changes via the flow" actually binding; record the exact settings in `project.md`.
- **Org-shared:** `changes-via-openspec` and `branch-per-change` are identical across projects — prime
  candidates for the copy-in `standards/` bundle (pinned + integrity-checked), so every repo inherits
  the same rule, while each projects its own LOCAL enforcer from the shared scenarios.
