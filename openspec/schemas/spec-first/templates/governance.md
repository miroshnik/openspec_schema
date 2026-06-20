# Governance — "all changes go through the flow" as enforceable standards

The strongest rule a spec-first repo can set is **process** itself: nothing reaches the
default branch except through a reviewed, gated, archived OpenSpec change. spec-first has a
home for exactly this — a **standard**. So governance rules are not a new mechanism; they are
ordinary standards (the schema dogfooding itself), enforced by the same gate, plus one
out-of-repo piece (branch protection) that no in-repo check can replace.

Drop these seed standards under `specs/governance/<name>/spec.md` (or `specs/_org/` to make
them org-shared and immutable — see the README). They follow `templates/standard.md`. The
enforcer named in each standard's `test:` is stack-specific; wire it into the gate
(`templates/gate.md` step 1 — they run in the project suite like any standard's enforcer).

> **Honesty about enforcement.** "No direct pushes to the default branch" is enforced by your
> git host's **branch protection / required checks**, NOT by anything in the repo. The repo
> ships the *checks that run on a PR*; the host ships the *rule that a PR is the only way in*.
> Both are required — document the branch-protection settings alongside these standards.

---

## Seed standard 1 — `changes-via-openspec` (the architectural one)

```
---
test:       test/governance/changes-via-openspec — a conforming PR passes; a src-only PR with no change fails. The enforcer is ci/changes-via-openspec — on a PR it asserts: (a) the diff adds a changes/archive/<date>-<id>/ tree, (b) every changed NON-GENERATED source path is referenced by that change's tasks.md or a spec (lockfiles/vendored/regenerated paths excluded), (c) the default branch is push-protected — asserted out-of-band by branch protection, NOT by this script
tier:       mechanical            # structural checks are mechanical; "diff matches intent" is a judge backstop
governs:    the default branch — every commit that lands on it
---

# Changes via OpenSpec

## Purpose
Generation is only trustworthy if nothing reaches the default branch outside the reviewed,
gated flow. This standard makes the OpenSpec change the ONLY path to the default branch.

## Decision Record
- **Status:** accepted
- **Context:** ad-hoc commits bypass specs, tests, and the gate — the whole trust guarantee leaks through that gap.
- **Decision:** the default branch accepts changes ONLY via an OpenSpec change merged through a PR; direct pushes are blocked at the host.
- **Consequences:** every change is specced, tested, gated, and archived; even a hotfix is one fast change. Trade-off: no out-of-band commits, ever.
- **Alternatives considered:** honor-system / docs-only → unenforced, drifts. CODEOWNERS → governs *who reviews*, not *flow-conformance*.

## Requirements

### Requirement: Every merge to the default branch is an archived OpenSpec change
The default branch SHALL accept commits only through a PR that archives exactly one OpenSpec
change, and direct pushes to it MUST be rejected.

#### Scenario: conforms
- **Given** a PR from `change/add-auth` whose diff archives `add-auth`
- **When** the gate runs
- **Then** the `changes-via-openspec` check passes.

#### Scenario: violates
- **Given** a PR that edits `src/` with no OpenSpec change in the diff
- **When** the gate runs
- **Then** it fails: "no OpenSpec change — author one with `openspec new change`."

## Rationale log
- **v1 (<date>):** mandatory. Narrow exceptions (bot dependency bumps) still go through a `chore/<id>` change, never a bypass.
```

---

## Seed standard 2 — `branch-per-change`

```
---
test:       test/governance/branch-per-change — a single-change branch passes both pre- and post-archive; two changes on one branch, or a name mismatch, fails. The enforcer is ci/branch-per-change — it asserts exactly ONE change is introduced by this branch — an active changes/<id>/ (pre-archive) OR a newly-added changes/archive/<date>-<id>/ in this PR's diff (post-archive) — AND its <id> == the branch name with the `change/` prefix stripped
tier:       mechanical
governs:    git branches / worktrees that hold OpenSpec changes
---

# One branch per change

## Purpose
A change is the unit of review and of isolation. Binding it to a branch (or git worktree) named
by its id keeps PRs, gates, and archives strictly one-to-one and reviewable.

## Requirements

### Requirement: A branch introduces exactly one change, named for it
Each OpenSpec change SHALL live on its own branch named `change/<change-id>` (id matching the
branch name), and a branch MUST NOT introduce more than one change — whether that change is
still active or already archived in the diff.

#### Scenario: conforms
- **Given** branch `change/add-auth` introducing exactly one change — `changes/add-auth/` before archive, or `changes/archive/<date>-add-auth/` in the PR diff after archive
- **When** the gate runs
- **Then** `branch-per-change` passes in both states.

#### Scenario: violates
- **Given** a branch introducing two changes, or `changes/add-auth/` on branch `feature/x`
- **When** the gate runs
- **Then** it fails, naming the mismatch.

## Rationale log
- **v1 (<date>):** worktrees count — `git worktree add ../add-auth -b change/add-auth`. The rule is name + singularity, not branch-vs-worktree.
```

---

## Seed standard 3 — `change-lifecycle`

```
---
test:       test/governance/change-lifecycle — a PR diff missing the archive fails; the gate-green and approval legs are host-enforced (required checks), not unit-tested here. The enforcer is ci/change-lifecycle — the in-repo script asserts ONLY that the change is archived in the PR diff; gate-green and ≥1 approval are enforced out-of-band by required-checks / branch protection (no in-repo fixture)
tier:       mechanical            # the in-repo half (archive present) is mechanical; gate-green + approval are host-enforced
governs:    the PR → archive → merge transition of every change
---

# Change lifecycle

## Purpose
Pin the end state so specs, gate, and archive cannot be skipped on the way to the default branch.

## Requirements

### Requirement: Gate-green, approved, and archived before merge
A change SHALL reach the default branch only when its gate is green, its PR is human-approved,
and `openspec archive` has run so the merged diff carries the updated `specs/`. A merge missing
any of the three MUST be rejected. The approval RATIFIES THE SPEC AS THE SOURCE OF TRUTH — a
functional capability's intent and the strength of its scenarios, not only any standards
CREATE/REVISE — because the gate proves consistency among projections but not the source's
correctness.

#### Scenario: conforms
- **Given** a PR whose gate is green, that is approved, and whose diff archives the change
- **When** merge is attempted
- **Then** it is allowed.

#### Scenario: violates
- **Given** a PR that is green and approved but did NOT run `openspec archive`
- **When** merge is attempted
- **Then** it is rejected: "archive the change before merge."

## Rationale log
- **v1 (<date>):** the check verifies the END STATE (all three present at merge), not a strict temporal order, so PRs can open early; the LAST commit before merge is typically `openspec archive`. If your host dismisses approvals on new commits, re-request after archiving.
- **v1 (note):** only the archive-in-diff leg is mechanically checkable in-repo; gate-green and approval are host-enforced (required checks + branch protection) with no in-repo fixture — same split as the honesty note at the top of governance.md.
```

---

## The change lifecycle, step by step

```
0. (brownfield) adoption mode set — see SETUP Step 0.
1. START   — `openspec new change <id>` and branch: `git worktree add ../<id> -b change/<id>`
             (or `git switch -c change/<id>`). Convention: change id == branch suffix; ONE change per branch.
2. AUTHOR  — proposal → specs → design → tasks (the flow). design DECIDES standards; specs AUTHORS them.
3. BUILD   — implement the tasks; run the GATE after each task group (continuously) until green,
             resolving/justifying every hard-local judge suspect.
4. ARCHIVE — `openspec archive <id>` on the branch → merges delta specs into `openspec/specs/`,
             moves the change to `changes/archive/`. Re-run the gate over the updated specs until green.
5. PR      — open the PR from `change/<id>`. CI runs the gate (soft judge posts suspects). The
             changes-via-openspec, branch-per-change, and change-lifecycle checks must pass.
6. REVIEW  — a human approves, ratifying any standards CREATE/REVISE. Branch protection requires
             approval + all required checks green.
7. MERGE   — squash-merge to the default branch. Direct pushes are blocked at the host, so this PR
             is the only way in — closing the loop with `changes-via-openspec`.
```

## Wiring (per project)

- **In repo (version-controlled):** the three enforcers named in the standards' `test:` bodies above
  (ci/changes-via-openspec, ci/branch-per-change, ci/change-lifecycle), run by the gate (suite, step 1).
  Implement once; they key off `changes/`, the branch name, and the PR diff.
- **At the git host (cannot live in the repo):** protect the default branch — no direct pushes,
  require a PR, require the gate's CI check green, require ≥1 approval. This is the half that makes
  "all changes via the flow" actually binding; record the exact settings in `project.md`.
- **Org-shared:** `changes-via-openspec` and `branch-per-change` are identical across projects — prime
  candidates for the org-standards bundle (`specs/_org/`), so every repo inherits the same, immutable rule.
