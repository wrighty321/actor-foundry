# 010 — Repair, fixture re-approval and the un-pause gate
Status: accepted — 2026-10-18, by the user (proposed the same day by the Kun pass on the architecture)
Supersedes: the un-pause gate of 003 and 004 (`paused` with a `repair_id`); adds to 002 and B3

## Context
- ADR 002's L1 compares `extract(source/*)` with `expected.jsonl` on pages saved at approval
  and "never refreshed except by a new approval". After a real site change the repaired parser
  must still reproduce the old rows from the old HTML **and** satisfy the anchors on the new
  HTML. That holds only while every parser stays bi-modal; the first field that moves into a
  JSON endpoint breaks it. architecture.md's own boundary test says a site change "touches
  `extract.py`, the fixture's anchors" — and only the user may touch the fixture (B3). The
  repair flow therefore had no path to green that the documents defined.
- ADR 004 un-paused when the control record showed `paused` with a `repair_id`. Nothing
  wrote `repair_id`: the check set it null on pause, and the repair workflow held no Apify
  token (ADR 001). As written, nothing ever un-paused automatically. Traced by the
  independent review, 2026-10-18.

## Decision
- A repair PR that cannot pass L1 against the approved `source/` ships a **fixture draft in
  the same PR**: new `source/` fetched from the live site at repair time, new
  `expected.jsonl`, updated `anchors`, `status: draft`, `fetched_on`. The PR body says so.
- **Merge deploys the code and does not un-pause.** The deploy un-pauses only when L1 and L2
  pass on the deployed build **and** `fixture.md` reads `status: approved` with `approved_on`
  on or after the repair commit's date. Otherwise the actor stays paused and Telegram says
  "fixture needs your approval: actors/<name>/fixture/fixture.md".
- The user's approval is a commit of their own that sets `status: approved` — the ruling on
  what "working" means now. It is distinct from the merge, which rules on the code.
- `repair_id` stays in the control record as information only; it gates nothing.

## Alternatives
- **Bi-modal parsers that satisfy old and new sources.** Rejected: unbounded complexity for
  the sake of a test that no longer describes the site.
- **Refresh `source/` automatically on repair.** Rejected: that is the builder writing the
  fixture, which the vision rules out.
- **Un-pause on merge alone.** Rejected: a merge is a ruling on code; whether the new rows are
  the right rows is the fixture owner's ruling.
- **Keep the `repair_id` gate and give the repair job an Apify token.** Rejected: widens the
  repair job's secrets to fix a gate that the fixture status answers better.

## Consequences
- Two taps after a site change — merge, then approve the fixture — accepted because they are
  different judgements and the second is the one the brief makes the user's.
- The deploy workflow reads `fixture.md` frontmatter; B3 gains the re-approval rule.
- A repair whose parser still passes the old L1 needs no fixture change and un-pauses on
  merge alone, since `status: approved` already holds.
