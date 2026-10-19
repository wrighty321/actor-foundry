# 004 — Approval and deployment: merging the pull request is the ruling; deploy runs on merge
Status: accepted — 2026-10-18, by the user
Amended 2026-10-18: the un-pause gate by 010; the "no required-PR rule" consequence by 011; the `listing-change` check now requires a `decisions.md` line carrying `#<PR number>` (architecture.md).
Amended 2026-10-18 (quality-score pass): `foundry lint` joins the PR CI job and the deploy step (014); the publish runbook names the Console steps the deploy only reads back (015).

## Context
- Ruled 2026-10-18: a maintainer repair that passes the fixture is deployed only on the user's
  ruling; escalations and requests go to Telegram; the maintainer never publishes.
- A GitHub Actions cron job cannot receive a Telegram reply. Something would have to poll the
  bot API, keep an update offset, and parse replies — a second system, with its own failure
  modes, for one click.
- Every write action should trace to the human who granted it (Kun pass, 2026-10-18); GitHub
  records who merged what, when.
- The user ruled on 2026-10-18 (architecture session): approve by merging on GitHub; Telegram
  carries the link.
- The first public listing, and any later change a buyer sees (README, input schema, price),
  still need a dated ruling in `context/decisions.md`; a code repair that changes none of those
  does not.

## Decision
- **Repair request** = a pull request opened by the repair workflow (ADR 001) plus one Telegram
  message: actor, structural cause, PR link, the fixture result on the branch (L1 and L2), and
  the sentence "Merge to deploy; close to reject".
- **Ruling** = merging that PR. Closing it rejects; the actor stays paused and the escalation
  stands.
- **`deploy.yml`** runs on push to `main` touching `actors/<name>/**`: `uv run pytest` for that
  actor (L1), build and vendor the `foundry` wheel (ADR 006), `apify push` the actor, run L1 and
  L2 against the deployed build, and — if the control record shows `paused` with a `repair_id` —
  `foundry unpause` (ADR 003). It appends a log line with the merge SHA and the merger
  (ADR 005). If L2 fails on the deployed build the actor stays paused and Telegram says so.
- **The deploy workflow never changes visibility, price, README-visible text or the input
  schema's limits**: `isPublic` is flipped by the user after the publish ruling; a PR that
  touches `README.md`, `.actor/input_schema.json` or `pricingInfos` is labelled `listing-change`
  by the workflow and needs a `decisions.md` line dated on or before the merge — the deploy step
  fails with a clear message if the line is missing.
- **Hand fixes** take the same road: a PR (or a direct push, which the convention discourages)
  to `main` runs the same deploy and un-pause.

## Alternatives
- **Telegram inline buttons polled by a workflow.** Rejected by the user's ruling; mechanically
  it adds a poller, an offset store and a reply grammar, and a missed poll silently delays a
  deploy.
- **A Telegram webhook bridged to `repository_dispatch`.** Rejected: a hosted component holding
  the bot token outside Actions.
- **A PR review approval (not merge) plus auto-merge.** Rejected: two clicks and an auto-merge
  setting to keep enabled; the merge itself is the cleaner signal.
- **Deploy from the repair workflow directly on "approve".** Rejected: the maintainer would
  then publish, which it never does.

## Consequences
- The GitHub mobile app joins Telegram as an operator surface; one tap approves.
- `main` cannot require pull requests for every push, because the check workflow commits log
  lines directly (ADR 005). Either exempt the Actions bot or keep the no-protection default
  with the convention that actor code arrives by PR. Decided for now: no required-PR rule;
  revisit when a second contributor exists.
- `APIFY_TOKEN` is needed by two workflows (check and deploy); the Telegram secrets by check,
  deploy and notify. "Scoped" means the smallest set of workflows that need it, not one.
- Commits us to: the PR as the unit of change for actor code, the `listing-change` check, and
  the deploy workflow as the only caller of `apify push`.
