# Add rate limiting to the public HTTP API

## Why

The public API has no rate limiting: a single client can exhaust upstream capacity and
run up unbounded cost, and there is no uniform way to shed abusive traffic. We add a
per-client token-bucket limiter AND a standard that makes "every public handler is
limited" a checked rule, so new endpoints cannot ship unlimited by omission. (≥50 chars.)

## What changes

- A `rate-limiter` FUNCTIONAL capability (a pure native OpenSpec spec): a `withRateLimit`
  middleware with token-bucket semantics (under-limit → allow; over-limit → 429 + `Retry-After`).
- A `rate-limit-policy` STANDARD (`governs:` public HTTP handlers) — a self-contained living
  record at `standards/rate-limit-policy.md`, enforced by an AST rule requiring every
  public-router export to be wrapped by `withRateLimit`.
- A migration: a codemod wrapping the 3 existing unwrapped public handlers.

## Affected surfaces & rollback

- **Surfaces touched** — design retrieves (by reading `standards/` directly) the records whose
  `governs:` covers these:
  - `src/routes/public/**` — the public router and its handler exports (newly governed by
    `rate-limit-policy`).
  - `src/middleware/**` — the new `withRateLimit` limiter.
  - `src/lib/eslint-rules/**` + eslint config — the new `require-rate-limit` AST rule.
  - Test tree — `test/rate-limiter/**` and `test/standards/**`.
- **Rollback** — the limiter is a single middleware wrapper and the policy is one lint
  rule. To roll back: revert the codemod (handlers unwrap cleanly), disable the
  `require-rate-limit` rule in eslint config, and remove the `withRateLimit` import. No
  data migration, no persisted state beyond the in-memory/Redis bucket store, which is
  ephemeral — dropping it simply resets counters. The functional `rate-limiter` spec is archived
  into `openspec/specs/`; a rollback change would author a `## REMOVED Requirements` delta rather
  than deleting the file. The `rate-limit-policy` standard is a plain git file under `standards/`
  (never archived); rolling it back is an ordinary commit deleting or amending that record (with a
  Rationale log entry).
