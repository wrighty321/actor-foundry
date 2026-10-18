# 008 — Pricing events: one `result` event per complete row, the default start event, nothing else
Status: accepted — 2026-10-18, by the user

## Context
- Ruled: every actor is pay-per-event with limited permissions, charged through
  `push_data(..., charged_event_name=...)`, priced under the incumbent for the same data, never
  below platform cost.
- Agents paying per request can run only actors that charge for events only (no "+ usage"),
  with limited permissions and no Standby (`context/research-2026-10-18.md` §1).
- Apify charges the synthetic `apify-actor-start` event automatically at a default $0.00005
  and recommends keeping that price; a second synthetic event, `apify-default-dataset-item`,
  charges per item written to the default dataset and can be removed in the Console
  (pay-per-event docs, 2026-10-18).
- When the buyer's spending limit is reached the SDK caps charging and writing without
  raising; `ChargeResult.event_charge_limit_reached` must be checked (research notes §1).
- Incumbent prices are tiered per buyer plan (`eventTieredPricingUsd`, observed 2026-10-18);
  the candidate card names the FREE-tier price as the comparison basis.

## Decision
- Each actor defines exactly one charged event, **`result`**, charged once per complete row by
  `foundry.charge.push_row(row)`, which wraps `Actor.push_data(row, charged_event_name="result")`,
  checks `event_charge_limit_reached`, and on limit persists state and exits cleanly with the
  status message "Spending limit reached after N results; nothing further was charged".
- `apify-actor-start` stays enabled at the default price. `apify-default-dataset-item` is
  removed, so a row is charged once (**assumed**: both synthetic and custom events would
  otherwise bill the same push; verified on the stub's first charged run).
- The `result` price per actor is set at publish from the compute measured on the fixture run
  (CU per 1k rows, proxies) against the incumbent's FREE-tier per-1k price, and is written on
  the candidate note and in the README. The maintainer never changes it.
- A row is "complete" when every field in the fixture's `fields` is present; incomplete rows
  are not pushed and are counted in the status message.

## Alternatives
- **Charge per page fetched.** Rejected: buyers cannot predict cost and pay for pages that
  yield nothing.
- **Tiered events by field richness.** Rejected: complexity before any evidence buyers want
  it; a second price to explain in the README.
- **Use the synthetic dataset-item event instead of `result`.** Rejected: it bills every
  dataset write, including rows we would rather not charge, and takes the "complete row" rule
  out of our code.
- **Pay per event + usage while calibrating.** Ruled out: lowers the quality score and excludes
  per-request agentic payments.

## Consequences
- Easier: one number to compare against incumbents, one line in every README, one wrapper to
  test for the limit path.
- Harder: the price must be set from a measurement, so publish day depends on the fixture
  run's usage reading.
- Commits us to: the `result` event name across actors, `foundry.charge` as the only path to
  `push_data`, and keeping the start event at Apify's default.
