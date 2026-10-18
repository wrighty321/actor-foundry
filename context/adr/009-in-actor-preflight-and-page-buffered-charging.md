# 009 — In-actor preflight and page-buffered charging
Status: accepted — 2026-10-18, by the user (proposed the same day by the Kun pass on the architecture)
Adds to: 002 (reuses L2 inside the actor), 008 (charging per page, not per row)

## Context
- The brief names silently wrong rows as the expensive case and expects the maintainer's
  daily run to notice them. Between a site change and that run — up to a day, and GitHub
  documents that `schedule` runs "can be delayed … some queued jobs may be dropped" at busy
  times (docs.github.com, read 2026-10-18) — every buyer run produces and charges rows.
- A markup shift that moves `title` into `company` yields rows that are **complete** by field
  count and wrong by content. The actor's only in-run guard as designed was "zero complete
  rows → fail", which such a shift never triggers.
- The structure check already exists: ADR 002's L2 (fields present and typed, anchors,
  `min_rows`) in `foundry.fixture`. Running it inside the actor against one page costs one
  fetch per run.
- ADR 008 charges on every `push_data`; a page that extracts half-wrong cannot be un-charged.

## Decision
- **Preflight.** After the control read (ADR 012) and before any push, the actor fetches the
  fixture's `live_pages[0]` (or reuses it when the buyer's input starts there), runs `extract`
  and asserts L2. On failure: `Actor.fail(exit_code=12, status_message="Site structure changed
  on <url>: <failed assertion ids>; nothing was charged")`. Exit code 12 is reserved for
  "preflight failed".
- **Page-buffered charging.** `foundry.charge.push_rows(rows)` replaces per-row push: for each
  fetched page, extract, compute the share of complete rows, and push the page's complete rows
  (charged) only when the share is at least `min_complete_share` (a fixture field, default
  0.9); below it, fail with the counts and nothing from that page is charged. Incomplete rows
  are never pushed.
- **The maintainer reads exit codes.** The daily check lists the actor's runs since its last
  line and counts exit code 12; any count above zero is escalated as a site change even if
  the check's own L2 passed (different page, different hour).
- **The receipt is free.** If the parked freshness-receipt bet (plan row B1) ever passes its
  check, the preflight result is the receipt: one KV write, no new component.

## Alternatives
- **Rely on the daily check alone.** Rejected: up to a day of charged wrong rows, longer when
  the schedule slips.
- **Run the maintainer hourly.** Rejected: twenty-four times the Actions minutes and still a
  window.
- **Keep the zero-rows guard only.** Rejected: field shifts keep rows complete.
- **Full L2 on every page.** Rejected: anchors are page-specific; per-page shape and share is
  the right test for pages the fixture never saw.

## Consequences
- One extra fetch per run at most; a fixture field `min_complete_share`; exit code 12.
- A preflight that fails because an anchor moved while the site is otherwise fine blocks runs
  until the user updates the anchor — accepted: that is the fixture doing its job, and the
  message names the anchor.
- `push_rows` is also the fix for per-row `push_data` throughput at thousands of rows.
- Commits us to: `foundry.fixture` being importable by actors (it already is, ADR 006) and the
  check reading run lists through the scoped token (ADR 012).
