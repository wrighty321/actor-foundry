# Project Plan — actor-foundry

> **LIVING** — rewritten by `/consolidate`, read by `/status`. Not the brief; not frozen.
> This is current state and near-term plan. Long-lived intent lives in `project-brief.md`.

<!-- PLAN:BODY:START -->
## Milestones
| # | Deliverable | Done when | Blocked on | Status |
|---|---|---|---|---|
| 1 | Maintainer loop against a stub actor on a stable site | The loop runs on its chosen schedule (GitHub Actions cron or always-on machine — decided here), checks a fixture, and writes one log entry per run for 7 consecutive days | — | not-started |
| 2 | Actor #1: hand-picked after an Apify Store gap check | Fixture authored by the user; builder PR passed adversarial review; ruling in `decisions.md`; published pay-per-event with limited permissions; maintainer green on publish day | 1 | not-started |
| 3 | Actor #2, same route | As milestone 2 | 2 | not-started |
| 4 | 30 unattended days | Maintainer log shows 30 consecutive daily runs across both actors with at least one repair or named escalation | 3 | not-started |
| 5 | Grok Bot watcher (deferred) | A daily routine reads Tranco deltas, Product Hunt / Hacker News launches and X, and hands over a ranked shortlist; no credentials for this repo on the Grok Bot computer | 2 and 3 earning | not-started |

## Risks & unknowns
| Risk | Known / Unknown | Mitigation, or cheapest test to resolve it |
|---|---|---|
| Chosen site sits behind anti-bot (Cloudflare, DataDome) and needs residential proxies, so the actor runs at a loss | Unknown | Plain `fetch` of three target pages before committing to a site; price from measured compute units on the fixture run |
| In-app scheduled tasks fire only while the app is open, so the maintainer cannot live there | Known | Milestone 1 picks GitHub Actions cron or an always-on machine, and proves 7 unattended days before any actor depends on it |
| Pay-per-event pricing below platform cost (negative profit) | Known | Weekly read of profit per actor in Apify Actor analytics; `apify-actor-start` synthetic event on; memory limits set |
| Nobody finds the actors on the Store | Unknown | Read paid and free user counts 30 days after each publish; promote per Apify's marketing checklist only if zero |
| Grok Bot shares one computer and its credentials across every bot on the account | Known | The watcher is read-only; the Apify token lives in this repo's `.env` and is never placed on the Grok Bot computer |
| A target site's terms forbid scraping, or the data turns out personal | Known | Human ruling before build, recorded in `decisions.md`; hard exclusion in the brief |
| The builder cannot handle a site unaided and the user finishes it by hand | Known | Expected for roughly half of early candidates; each fix lands in the actor template so the next build absorbs it |

## Resourcing
- Appetite (from feasibility §B4): multi-week — two actors plus the maintainer loop, about 4–6 weeks
- Time budget: roughly 6–8 hrs/week around a fixed 20 hrs/week of lectures, through December 2026
- Skill gaps + how closed: Apify SDK and Crawlee for Python (read the docs this session; closed by building the stub in milestone 1); anti-bot handling (closed per site, or the site is dropped)
- Costs: one-time — none; recurring — none new (existing Claude, Cursor/Grok Bot and Apify plans; the Creator plan's $500 prepaid usage covers testing for 6 months); per-use appetite (§B4b): existing plans only, no new recurring spend until an actor earns; visibility wanted — profit per actor from Apify analytics weekly, compute units per maintainer run in its log
<!-- PLAN:BODY:END -->
