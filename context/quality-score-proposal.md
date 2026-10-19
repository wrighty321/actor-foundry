# Proposal — building for Apify's quality score

Status: **integrated 2026-10-18** — drafted from the user's note *Apify Actor Quality Score
Playbook* (2026-10-18), reviewed by an independent Kun-lens pass the same day, revised to the
pass's findings, and written into ADRs 014 and 015, `architecture.md`, the playbook, the plan
and `build-plan.md` Stage U3. The original draft's items A–N are kept below with their final
disposition so the reasoning is traceable; the ADRs are the binding text.

## What the score changes about this repo

The score is a 0–100 number Apify computes per public actor from eight categories (docs read
2026-10-18); Store search and the MCP `search-actors` tool rank by roughly the same signals.
Three mechanics bear on designs already accepted here:

1. **Apify's daily test** runs the prefill input and expects `Succeeded` with a non-empty
   default dataset within 5 minutes; 3 failed days → `under maintenance`, +14 → notice, +14 →
   `deprecated`. A break we do not repair within about 72 hours earns the label whatever we do;
   one never repaired is deprecated by Apify around day 31. A target-site break that passes most
   test runs in the next 7 days heals on its own — **unless the actor is paused**, which is why
   the pause is now narrowed (ADR 015 §1).
2. **Reliability and feedback weigh most.** Honest failures (exit 10/12) lower the success rate;
   the brief chose honesty, so the lever is repair speed. At twenty buyer runs a day, a five-day
   break costs roughly fifteen points of 30-day success rate until it rolls off.
3. **Congruency is scored** across title, description, README, input schema and dataset
   schema. The fixture's `fields` is already the vocabulary; ADR 014 generates the dataset schema
   from it and lints the rest against it.

## What the Kun pass changed (2026-10-18)

- **Pause narrowed** (M1): an L2 failure confined to `live_pages[0]` is escalated, not paused —
  the preflight already blocks buyers charge-free, and a pause only locks in the daily-test
  failure. The draft's claim that "no existing component works against the score" was wrong.
- **Apify's verdict read, not inferred** (M2): `notice` and `isDeprecated` from `/acts/{id}`,
  not a `/3` counter built from `meta.origin == TEST` runs whose visibility is unverified.
- **KYC is a gate** (M3): default Store search excludes actors from developers without KYC
  (observed totals 90,859 vs 91,585 with `includeUnrunnableActors=true`).
- **`default` as well as `prefill`** (M4): `prefill` is UI-only.
- **Generate only the dataset schema; lint the rest** (M5).
- **Saturation is not the Store `search=` total** (S1): "google maps" 3,240, "linkedin jobs"
  2,023, "planning applications" 502 — every real query is in the 50+ band. Replaced by the
  agent's count of actors whose title names the site, from the snapshot.
- **No word-count or keyword lint** (S2): metric theatre.
- **No `listing.yaml` writer** (S3): a record in the repo, Console edits at publish.
- **Never delete and never deprecate** (S4): `isDeprecated` is in the Update Actor body.
- **Console Monday is the existing weekly step, extended** (S5); no issues/reviews API exists
  (404s and no path in the OpenAPI spec, 2026-10-18).
- **Input validation merges into B15** (S6). **No trust-ladder language for an SLA** (S7).
- **Deprecated actors enter the weak sample by id diff; the snapshot header records its query
  parameters; `includeUnrunnableActors=true` in the pull** (S8).
- **Deferred**: tiered discounts until the first paid run (D1); task examples beyond one until
  14 clean daily tests (D2); Rising Star as a recorded `badge`, not a target (D3); the `/31`
  counter (D4).

## Open decisions (yours)

1. **Tiered discounts** — deferred to the first paid run by the pass; the schema has five tiers.
   If you want them at launch anyway, say so and ADR 015 §8 changes.
2. **Task examples** — the pass: one, after 14 clean daily tests; your note: three on day one.
   The platform refuses tasks before the actor is public either way.
3. **Co-author lines** on this repo's planning-session commits — the OS rule says never; the
   harness adds one. Your ruling (ADR 016).
4. **Preflight honesty** — kept; the success-rate arithmetic is stated in ADR 015.

## The items, with their final disposition

| Item | Draft | Disposition |
|---|---|---|
| A | Fixture generates listing vocabulary | **Kept, narrowed**: generate the dataset schema only; lint README output keys and input descriptions (ADR 014 §1, §3) |
| B | Prefill = fixture live check | **Kept, extended**: `prefill` and `default` (ADR 014 §2) |
| C | 72-hour clock in the maintainer | **Changed**: `notice` read from the API; one sentence on the clock; pause classes narrowed; day-14 ruling point (ADR 015 §1, §2, §7) |
| D | Input schema validation | **Merged** into B15 plus one lint rule (ADR 014 §3) |
| E | Listing fields applied by deploy | **Changed**: `listing.md` record; Console at publish; deploy reads back only (ADR 014 §4) |
| F | README contract | **Kept, trimmed**: structural lint, no word count or keywords (ADR 014 §3) |
| G | Pricing tiers and cost example | **Split**: cost example linted now; tiers deferred (ADR 014 §3, ADR 015 §8) |
| H | Three task examples at launch | **Deferred**: one, after 14 clean tests (ADR 015 §8) |
| I | Console Monday | **Changed from Add to Change**: the existing weekly step, extended; no issues API (ADR 015 §5) |
| J | Candidate card launch inputs | **Kept, corrected**: title/slug; saturation as a hand count from the snapshot, not the search total; Demand ≥ 2 in the 1–5 band justified by Base = Demand × Gap, not an anecdote (`candidate-card.md`) |
| K | Wiki pull maintenance signals | **Kept, extended**: `isDeprecated` by id diff, `createdAt`, `modifiedAt`, `badge`, `includeUnrunnableActors=true`, query parameters in the header (`wiki/README.md`) |
| L | Publish request as launch checklist | **Kept, extended**: KYC date; publish runbook; the Rising Star window reframed as buyer service (ADR 015 §3, §4) |
| M | Hard limits | **Kept, corrected**: never deletes, never deprecates, never the toggle (ADR 015 §6) |
| N | Remove nothing | **Corrected**: the pause-on-any-L2 rule did work against the score and is narrowed; the two merges stand |

## What this does not fix
- Honest failures cost success rate; the arithmetic is in ADR 015.
- History of success starts at zero; actor #2 benefits from #1.
- The weights are unpublished and the score is relative; Console Monday is how that is seen,
  and the pass's warning stands: a number on a dashboard gets optimised — read it monthly for
  trend, not weekly for action, once the first month is past.

## Verification done
- Eight categories; daily test rule and 3/14/14; dataset view required for tasks; `RunOrigin`
  includes `TEST`: Apify docs and API reference, 2026-10-18 (this session).
- `notice: NONE` and `isDeprecated` public on `/acts/{id}`; KYC exclusion text and totals; no
  issues/reviews path; Update Actor body fields; five pricing tiers; `badge: risingStar` seen:
  the independent reviewer's live checks, 2026-10-18.
- Store `search=` totals 3,240 / 2,023 / 502: this session, 2026-10-18.
- Unverified: `actor.json` carrying `title`/`description` through `apify push` (stub week);
  owner run-list visibility of tester runs (stub week); the Rising Star mechanics (recorded, not
  relied on).
