# 003 — Pause mechanism: a kill-switch record the actor reads first, plus a flagged description
Status: superseded by 012 — 2026-10-18 (was accepted 2026-10-18, by the user)
Superseded by 012 on 2026-10-18 (the kill-switch record, exit codes and read-back carry forward; the automatic `description` write does not). Un-pause gate superseded by 010.

## Context
- Ruled 2026-10-18: an actor the maintainer cannot repair is paused with a status message,
  never left running broken, never unpublished.
- The Store `notice` field (`UNDER_MAINTENANCE`) is not in the Update Actor request body:
  absent from the Python client's `UpdateActorRequest`, from the JS client's
  `ActorClient.update` field list, and present only in the response schema of the fetched
  `api/v2/act-put` page (three reads, 2026-10-18). A `notice`-based pause would need the
  Console and could not run unattended.
- Update Actor does accept `description`, `seoDescription`, `title`, `isDeprecated`,
  `isPublic`, `pricingInfos`, `actorPermissionLevel` (client references, 2026-10-18).
- A named key-value store can be opened from inside an actor with
  `Actor.open_key_value_store(name=..., force_cloud=True)` (Python SDK reference, 2026-10-18).
- The synthetic `apify-actor-start` event is charged automatically by Apify at a default
  $0.00005 per start and must not be charged in code (pay-per-event docs, 2026-10-18 —
  documented). A paused run therefore costs the buyer at most that.
- The brief's failure posture: a run fails loudly with a status message, never a silent
  partial result; "a free-tier limit that looks like a crash" and "plausible rows that are
  stale or partial without saying so" are both anti-examples.
- **Assumed:** failed runs count against the actor's Store quality score as well as in
  `publicActorRunStats30Days`.

## Decision
- A named key-value store **`actor-foundry-control`** in our account holds one record per
  actor, key `<actor-name>`:
  `{ "paused": bool, "since": iso-date, "reason": str, "repair_id": str|null,
     "original_description": str|null, "last_check": iso-datetime }`.
- Every actor's `main()` reads its record **before** processing input and before any
  `push_data`. If `paused` is true it calls
  `Actor.fail(exit_code=10, status_message="Paused by the maintainer since <since>: <reason>.
  No results were produced or charged. Status: <repo url>")`. Exit code 10 is reserved for
  "paused".
- If the control record cannot be read after three retries with backoff, the actor fails with
  `exit_code=11` and the message "Maintainer control store unreachable; refusing to run
  without a health check" — correctness over availability, per the brief.
- The maintainer pauses by writing the record **and** prefixing the actor's `description`
  with `PAUSED <date>: <reason> — ` via Update Actor, storing the original description in the
  record. Both writes are read back (`GET` the record, `GET /v2/acts/{id}`) before the pause is
  reported as done.
- Un-pause happens two ways, both through `uv run foundry unpause <actor>`: by the deploy
  workflow after a merged repair passes L1 and L2 on the deployed build, or by the user after a
  hand fix. It clears `paused` and restores the description, and reads both back.

## Alternatives
- **`notice: UNDER_MAINTENANCE` through the API.** Not possible unattended (above).
- **`isDeprecated: true`.** Rejected: says the wrong thing to buyers and may hide the actor
  from Store search — functionally the unpublish the ruling forbids.
- **`isPublic: false`.** Ruled out 2026-10-18: loses users and ratings.
- **Succeed with zero rows and a status message.** Rejected: a success with no data is the
  brief's "plausible partial result" and reads as a crash to anyone not reading the message.
- **Fail every fetch inside the actor.** Rejected: opaque; no reason reaches the buyer.
- **Change the price to zero while paused.** Ruled out: the maintainer never changes a price.

## Consequences
- The buyer of a paused attempt pays the default start fee (≤ $0.00005) and sees an honest
  failure; failed runs accumulate in the 30-day stats while paused — accepted as the cost of
  honesty, reduced by the description flag cutting attempts.
- One extra key-value read per run (negligible); every actor shares the same first ten lines
  (ADR 006's `foundry.control`).
- A control-store outage stops every actor loudly rather than letting a paused one run. If
  that proves too brittle, the first evidence of it goes into a new ADR, not a quiet relaxation.
- Commits us to: the control store name, the record schema, exit codes 10 and 11, and
  read-back after every pause or un-pause.
