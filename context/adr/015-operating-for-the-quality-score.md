# 015 — Operating for the quality score: pause only what the preflight cannot see, read Apify's verdict, gate publish on KYC
Status: proposed (quality-score pass and its Kun pass, 2026-10-18)
Supersedes: the pause trigger of 012 (pause on any L2 failure); amends 013 (status page, Monday summary), 010 (fast path), 004 (publish runbook), 008 (tiers deferred)

## Context
- Apify's daily test and its escalation: 3 consecutive failed days → `under maintenance`
  (ranks lower; also settable by a Console toggle with the same penalty), +14 → notice, +14 →
  `deprecated` (not searchable). A target-site break that passes most test runs in the following
  7 days is healed automatically; a rebuilt healthy actor is seen within 24 hours
  (docs.apify.com/actors/publishing/test and /actor-status, read 2026-10-18).
- After ADR 009 every buyer run already blocks itself on the `live_pages[0]` assertions, with
  zero charges, and passes again the moment the site does. A pause for that class protects no
  buyer; it only makes every test run fail until a human un-pauses, which defeats the 7-day
  self-heal. ADR 012 paused on any L2 failure.
- `notice` and `isDeprecated` are public on `GET /v2/acts/{id}` (observed `notice: "NONE"`,
  2026-10-18). Whether the owner's run list shows runs Apify's tester started is unverified; the
  repo already leans on run-list visibility for ADR 009's exit-12 count.
- Default Store search excludes actors "from developers who haven't passed KYC" and
  full-permission actors without a large user base (`includeUnrunnableActors` parameter text;
  totals 90,859 without the flag vs 91,585 with it, 2026-10-18). The plan's risk row said KYC was
  assumed before milestone 2 and unconfirmed. Without it actor #1 is invisible to the default
  search and to `search-actors`, and the kill criterion cannot be met.
- No API for actor issues or reviews exists as of 2026-10-18: `/v2/acts/{id}/issues`,
  `/reviews` and `/v2/actor-issues` return 404, and the OpenAPI spec (131 paths) has no path
  containing issue, review, quality or badge — two reads of different shape, so the absence is a
  state for that date, not a reading.
- `isDeprecated` **is** in the Update Actor body, so the unscoped token in deploy could set it.
- Success-rate arithmetic: at twenty buyer runs a day, a break repaired on day 5 leaves about
  100 failed against about 600 succeeded runs in the 30-day window — roughly 85 % until it rolls
  off around day 35. Honest exits cost that; the brief chose honesty.

## Decision
1. **Pause classes.** The daily check pauses only for: an L1 failure (parser regression the
   preflight cannot see); an L2 failure on a page the preflight does not cover (buyers exposed
   to shifted fields there); fetch-error two days running; control-store or charge faults. An L2
   failure whose failed assertions all sit on `live_pages[0]` is **escalated, not paused** — log
   line `action: escalated (preflight guards)`; the actor fails loud and charge-free on its own
   and heals when the site does. Deploy has no un-pause step for that class.
2. **Apify's verdict is read, never inferred.** The check reads `notice` and `isDeprecated` from
   `GET /v2/acts/{id}` and writes them to `maintainer/STATUS.md` and the log line. One Telegram
   sentence states the clock — "Apify labels this actor under maintenance after 3 failed daily
   tests" — with no `/3` or `/31` counters of our own. Counting exit-12 runs stays an ADR 009
   item the stub week verifies.
3. **KYC is a publish precondition.** The publish request carries "KYC passed on <date>"; the
   plan's risk row becomes a gate on milestone 2.
4. **A publish runbook** (`wiki/playbook/publish-runbook.md`) names the user's Console steps in
   order after the ruling: set the `result` price from the fixture run's measured compute units;
   set SEO title and description and categories; confirm limited permissions; flip `isPublic`;
   record every value in `actors/<name>/listing.md`; after the first daily test passes, publish
   one task example framed as a user goal with dataset view `overview`; for the first 14 days,
   reply to any issue the same day. Deploy reads back price, permission level and `notice`.
5. **The weekly reading.** The plan's existing manual weekly profit step becomes **Console
   Monday**, about fifteen minutes from the Monday after publish: profit per actor, the quality
   score's trend (not its recommendations, which linger after fixes), open issues, new reviews
   — one dated row per week in `wiki/first-party/<actor>.md`. The Monday Telegram summary lists
   it as due. Nothing automates issues or reviews, because nothing can.
6. **Hard limits extended.** The maintainer never deletes an actor and never deprecates one —
   `isDeprecated` is a human ruling and deploy never sends it — and never uses the maintenance
   toggle (Console-only in any case).
7. **The repair fast path** (amends 010). The fixture draft is approved on the PR branch — the
   user's own commit on that branch sets `status: approved` — so one merge deploys and
   un-pauses. A repair PR gets the pinned adversarial review like any PR touching `src/`. The
   72-hour window is operator time, not a trust-ladder rung; Apify's day-14 notice is a ruling
   point: repair, or let deprecation happen. The tripwire that matters is the vision's "more than
   2 hours a week on repairs the maintainer should have made".
8. **Deferred, stated here so they are not re-proposed.** Tiered discounts (Apify's schema has
   five tiers, `FREE` to `PLATINUM`; one `pricingInfos` change after the first paid run, no notice
   period while there are no paying users). Task examples beyond one until the actor has passed
   14 daily tests. Rising Star: the pull records `badge` (observed `risingStar` on one Store item);
   nothing chases it.

## Alternatives
- **Pause on every L2 failure (ADR 012 as written).** Rejected: locks in the daily-test
  failure and protects no buyer the preflight does not already protect.
- **Count `meta.origin == TEST` runs to infer Apify's clock.** Rejected: unverified visibility,
  and an estimate of a clock we do not own when the label itself is public.
- **Succeed with zero rows to protect the success rate.** Rejected again: the brief's anti-
  example, and it fails Apify's non-empty-dataset test anyway.
- **Automated issue triage.** Impossible without an API; a Store-page scrape of our own listing
  for a weekly number is not worth a component.
- **Tiers and three task pages on day one.** Deferred: the percentages have no evidence, tasks
  cannot exist before publish, and the only source for their effect is one vendor post.

## Consequences
- Fewer pauses; `maintainer/STATUS.md` gains a `notice` column; a site-change repair can be a
  single merge.
- Honest failures still cost success rate, and the arithmetic above says how much.
- The user has one runbook and one fifteen-minute weekly step; KYC must be done before
  milestone 2 publishes.
- ADR 012's record schema, exit codes, read-back and scoped token carry forward unchanged.
