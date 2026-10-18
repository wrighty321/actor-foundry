# 012 — Pause: kill switch only, with a scoped token in the daily check
Status: accepted — 2026-10-18, by the user (proposed the same day by the Kun pass on the architecture)
Supersedes: 003 (its kill-switch record, exit codes and read-back are carried forward unchanged)

## Context
- ADR 003 paused by writing the control record **and** prefixing the actor's `description`
  through Update Actor. The second write forces an **unscoped account token** into the most
  frequent workflow: Apify's scoped tokens cannot create or modify actors but can be limited
  to specific resources such as one key-value store (Apify integrations docs, read by the
  independent review 2026-10-18 — documented; the page did not load for this session, so the
  claim is carried at the reviewer's reading until the first scoped token is created).
- The description write has a restore race: a description the user edits in the Console while
  paused is overwritten on un-pause from a stale copy.
- What the flag buys is a warning to buyers who read the listing before running. Every buyer
  who runs already gets the exit-10 message, and with ADR 009 no wrong row is sold either way.
- The effect of a flagged description on Store ranking is unknown (architecture.md).

## Decision
- **Pause = the control record only.** `actor-foundry-control`, key `<actor-name>`:
  `{ "paused": bool, "since": iso-date, "reason": str, "repair_id": str|null,
     "last_check": iso-datetime }` — `original_description` is dropped. Every actor reads it
  first; `paused` → `Actor.fail(exit_code=10, …)`; unreadable after three retries →
  `exit_code=11`. Read back after every write. All as ADR 003.
- **The daily check holds `APIFY_TOKEN_SCOPED`**: read actors and runs, write the
  `actor-foundry-control` store, nothing else. The unscoped `APIFY_TOKEN` lives only in the
  deploy workflow (it is the one that runs `apify push`) and in the user's local `.env`.
- **Flagging the listing is manual.** `uv run foundry flag <actor>` and `unflag <actor>`, run
  by the user with the unscoped token when a pause lasts beyond a few days: prefix the
  `description`, store the original locally in the command's output, and on `unflag` refuse to
  restore unless the current description equals prefix + original (otherwise print both and
  stop).
- The `reason` written to the record and sent to Telegram is composed only from fixture field
  names, anchor ids and HTTP codes — never from fetched page content (it reaches the Store
  status message).

## Alternatives
- **Keep the automatic description write (ADR 003).** Rejected: unscoped token daily, restore
  race, and a benefit the preflight and exit message already deliver.
- **`notice`, `isDeprecated`, `isPublic`, price change, zero-row success.** Rejected for the
  reasons in ADR 003, which stand.

## Consequences
- The daily job's credential blast radius shrinks to one key-value store.
- Buyers who only read the listing are not warned unless the user flags by hand — accepted.
- One more token to create (the scoped one), named in `.env.example`.
- If the scoped token turns out unable to list runs (needed by ADR 009's exit-code count),
  that is the first thing the stub week checks, and the fallback is a second scoped token
  rather than the unscoped one.
