# 002 — Fixture semantics and format: two layers, user-marked anchors, approval in frontmatter
Status: accepted — 2026-10-18, by the user
Amended 2026-10-18: re-approval after a repair by 010; L2 reused as an in-actor preflight and the `min_complete_share` field by 009; L2's fetch runs as an Apify run of the deployed build (architecture.md).

## Context
- The fixture is the definition of "working". The brief names silently wrong rows as the
  expensive failure and makes the fixture user-owned; the builder never writes tests against
  it (vision, Kun pass).
- A fixture that compares only against a **saved** page is a parser regression test: it can
  never see the live-site change the project exists to catch. A fixture that compares full rows
  against the **live** site produces a false alarm the first time a price, count or listing
  moves — on most target sites, daily.
- The maintainer check must stay under 1 compute unit and should fetch at most a few pages.
- Today nothing records that the user approved a fixture; the maintainer cannot tell a draft
  from an approved one (upgrade assessment U2, 2026-10-18).
- A match is a candidate, not a finding (vault `knowledge/a-match-is-a-candidate-not-a-finding.md`):
  any comparison must name what it excludes, or it will one day count the wrong thing.

## Decision
`actors/<name>/fixture/` holds:
- `source/` — the fixture pages saved once at approval (HTML or JSON response, one file per
  page), never refreshed except by a new approval.
- `expected.jsonl` — the rows the user approved, one JSON object per line, extracted from
  `source/`.
- `fixture.md` — frontmatter: `status: draft | approved`, `approved_on`, `source_urls`,
  `fetched_on`, `live_pages` (the one to three URLs the daily check fetches), `min_rows`,
  `fields` (name and type per output field), `anchors` (a list of `field`, `kind`
  (`exact | present | pattern | range`), `value`, `page`) — the values the user marks as stable.

Two check layers, both run daily:
- **L1 — parser regression.** `extract(source/*)` must equal `expected.jsonl` exactly. Offline,
  deterministic, held by a committed pytest. A failure here is a code regression.
- **L2 — live structure.** Fetch `live_pages` with the actor's own fetcher, run `extract`,
  assert every field in `fields` is present with its type, row count ≥ `min_rows`, and every
  anchor holds. L2 never asserts a volatile value. A failure here is a site change (or a
  fetch block, which the HTTP status distinguishes).

**Pass** = L1 and L2 pass. The maintainer's structural cause is the list of failed assertions.

**Approval** is machine-readable: the builder commits `fixture.md` with `status: draft`; the
user sets `status: approved` and `approved_on` in a commit of their own (the commit author is
the record). `build-checklist.md` B3 and the maintainer both require `status: approved`.
`extract` is a pure function from a saved response to rows, separate from fetching, so L1 runs
with no network.

## Alternatives
- **Saved page only.** Rejected: blind to the live change that is the product's reason to exist.
- **Live page with full row equality.** Rejected: false alarms on any data movement; would never
  pass on listings, prices or counts.
- **Live page with numeric tolerances.** Rejected for now: thresholds are a second fixture nobody
  owns; explicit anchors say what is stable and why. Revisit if anchors prove too coarse.
- **Builder-written assertions.** Rejected (vision): a second copy of the builder's reading of
  intent cannot say whether code or test is wrong.
- **Approval as a `decisions.md` line only.** Rejected: the maintainer cannot read it reliably;
  the frontmatter is the machine record and `decisions.md` stays the human record of the
  publish ruling.

## Consequences
- Easier: a deterministic daily check; fixture changes are reviewable diffs; L1 is an ordinary
  test the stack already runs.
- Harder: anchors need the user's judgement per site. A site with no stable anchor is a signal
  against the candidate — the candidate card's "Suggested fixture page" should name likely
  anchors so this is seen before approval.
- Commits us to: `extract(response) -> rows` as the actor's core function, fetched pages as
  files, JSON Lines for rows, and three fixture files per actor the user can read in Obsidian
  or any editor.
