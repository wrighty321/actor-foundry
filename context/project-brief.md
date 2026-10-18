# Project Brief — actor-foundry

> **FROZEN** — original intent as of 2026-10-18. Do not edit.
> When the direction changes, record it in `decisions.md`; this file never moves,
> so drift can be measured against it.

## The problem it kills
Building a web scraper is cheap; keeping it alive is not. A scraper portfolio on Apify Store
stops compounding at the number of actors one person can repair by hand, because every target
site changes its layout eventually and a broken actor costs ratings, refunds and the Store
quality score. This repo kills that manual upkeep: each actor ships with a fixture that says
what "working" means, and a machine loop checks, repairs or escalates against it, so the
portfolio grows at the speed of the loop rather than the speed of the operator's hands. It is
also the first concrete test of the hands-off-portfolio thesis behind telos P12.

## Telos trace
None — accepted knowingly. Commercial tinkering under R2; the income chain (X13) is retired and
this becomes a goal only by promotion. Recorded as a signal against building, overruled on
2026-10-18 because the cost is bounded (existing plans only) and the maintainer loop is worth
proving regardless of revenue.

## Done looks like
- Two actors are public on Apify Store, each under the pay-per-event pricing model with limited
  permissions, listed under the user's developer account.
- Each published actor has a fixture directory in this repo (saved page or response plus
  expected rows) that was authored or approved by the user, and the maintainer check passes
  against it on the day of publish.
- The maintainer loop has run unattended for 30 consecutive days with one log entry per run, and
  within that window has either repaired a real break (fixture failing, then passing after an
  automated change) or escalated one with a named cause.
- Every publish to the Store is preceded by a dated ruling in `context/decisions.md` and a
  pull request that passed an adversarial review separate from the author session.
- At least one run of any actor has been paid for by someone other than the user (visible in
  Apify's Actor analytics as a paid user).

## Boundary condition
The product is actors sold through Apify Store. If what is being sold becomes bespoke scraping
for named clients, a scraping service or SaaS outside Apify, or a data product resold from the
scraped output, this is a different project.

## Out of scope
Hard exclusions — changing one of these makes it a different project:
- Scraping personal data, or any site behind a login wall.
- Publishing or updating a Store listing without a human ruling recorded in `decisions.md`.
- Scraping-as-a-service for named clients (see boundary condition).

Deferred — sequencing choices, expected to be revisited:
- The Grok Bot site watcher (feeds and X, a daily shortlist). First hit in the walkthrough: when
  choosing actor #3. What happens then: the user picks by hand after an Apify Store gap check,
  exactly as for actors #1 and #2. Revisit only once two actors are earning.
- Standby-mode actors and an MCP endpoint (`actor.json` `/mcp` path). First hit: never in the
  first two actors, which are batch scrapers. Revisit if a buyer asks for real-time access.
- A second Apify account for isolation. First hit: never while the Apify token stays in this
  repo's `.env` and off the Grok Bot computer.

## Original stack intent
- Python, Apify Python SDK, Crawlee for Python; one actor per directory under `actors/<name>/`
  with its fixture under `actors/<name>/fixture/`.
- Apify CLI (`apify push`) for deploys; pay-per-event charged through `push_data` with a
  `charged_event_name`; state persisted on the `PERSIST_STATE` event.
- Claude Code for the builder and the maintainer, in this repo. Where the maintainer's schedule
  runs (GitHub Actions cron or an always-on machine) is decided in milestone 1; in-app
  scheduled tasks are not it, because they fire only while the app is open.
- Grok Bot, later, for the read-only watcher routine; it holds no credentials for this repo.

## Quality intent — binding, not testable
- Reference anchors: Apify's own first-party actor pages (input form, README that states price
  per 1k results and free-tier limits up front); Kun Chen's `no-mistakes` ethos — a change is
  done when an evidence-backed review says so, not when the author says so.
- Anti-examples: an actor that returns plausible rows that are stale or partial without saying
  so; a README that oversells coverage; a "free tier limit reached" that looks like a crash.
  Each would be functionally correct by some reading and is still rejected.

## Integrity contract
- Data criticality: irreplaceable — nothing (fixtures live in git; Store data is regenerable).
  Regenerable at a cost — each fixture (one human session to re-author). Disposable — run
  outputs, logs, scratch datasets.
- Failure posture: an actor run fails loudly with a status message, never a silent partial
  result; the maintainer degrades and escalates — it may open a fix, it never publishes one the
  fixture did not pass; the builder stops on any unverified step.
- Correctness-critical surfaces: the fixture match (silently wrong rows are the expensive case;
  noticed by the maintainer's daily run) and profit per actor (negative-profit pricing is
  silent; noticed weekly in Apify's Actor analytics, cost per 1k results against price).
- Concurrency reality: user runs happen in parallel, tens per day at first; the maintainer runs
  once per actor per day; the builder is serial, one actor at a time.
