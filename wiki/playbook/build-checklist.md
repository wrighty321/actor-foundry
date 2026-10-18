# Build checklist

The rules the builder follows for every actor. This page is playbook: it changes only through
a pull request the user approves. A new rule cites the synthesis page and evidence that
justify it (see the wiki README). Every builder pull request lists the IDs below it applied
and says why any did not apply, so the adversarial review can check each one.

IDs are stable: never renumber or reuse one; retire a rule by marking it retired.

## Before code
- **B1** Build only from a candidate note with `status: approved` **and** a dated ruling in
  `context/decisions.md`. Either missing: stop.
- **B2** Read [[what-makes-a-good-actor]], [[what-makes-a-bad-actor]] and [[common-mistakes]],
  and the category's page under `synthesis/topics/` if one exists.
- **B3** The fixture lives in `actors/<name>/fixture/` (saved page or response plus expected
  rows). The builder drafts the expected rows from the approved fixture page; nothing counts
  as the fixture until the user approves it. The builder never writes tests against the
  fixture and never edits it after approval.

## Money and platform
- **B4** Pay-per-event. Charge through `Actor.push_data(item, charged_event_name=...)`.
- **B5** Check `event_charge_limit_reached` on the returned `ChargeResult`; when the user's
  spending limit is reached, stop cleanly with a status message — never keep scraping unbilled.
- **B6** Save progress on the `PERSIST_STATE` event, so a migrated run resumes instead of
  starting over and charging the same result twice.
- **B7** Limited permissions (Python SDK v3.0.0 or later): touch only the default storages and
  what the user passes in.
- **B8** Set memory limits, and enable the `apify-actor-start` synthetic event.
- **B9** Charge for events only — no "+ usage" pricing in a published version, and no Standby
  mode (both exclude the actor from per-request agentic payments).
- **B10** Price under the incumbent for the same data, and above platform cost at the compute
  measured on the fixture run.

## Failure behaviour
- **B11** Fail loudly: a run that cannot produce correct rows ends with a status message
  saying why, never a silent partial result.
- **B12** Stale or partial data is labelled as such in the output and the status message.
- **B13** A free-tier limit exits gracefully with a status message that names the limit; it
  must never look like a crash.

## Listing
- **B14** The README states the price per 1k results and the free-tier limits up front, shows
  real example output, and does not claim coverage the fixture does not show.
- **B15** The input schema states the same limits, and its defaults produce a useful first run.

## Code
- **B16** Python with `uv` (environment and lock), `pytest`, `ruff` format and lint; type hints
  on public functions; plain functions over class hierarchies; async only where the Apify SDK
  requires it.
- **B17** Open source.
- **B18** Any fix made by hand during a build goes back into the actor template, so the next
  build absorbs it.
- **B19** Stop on any step that cannot be verified, and say which.
