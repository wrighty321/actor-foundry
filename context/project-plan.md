# Project Plan — actor-foundry

> **LIVING** — rewritten by `/consolidate`, read by `/status`. Not the brief; not frozen.
> This is current state and near-term plan. Long-lived intent lives in `project-brief.md`.

<!-- PLAN:BODY:START -->
## Milestones
| # | Deliverable | Done when | Blocked on | Status |
|---|---|---|---|---|
| 1 | Maintainer loop against a stub actor on a stable site | The loop runs on GitHub Actions cron (ruled 2026-10-18), checks a fixture, escalates to Telegram on a deliberately broken fixture, and writes one log entry per run for 7 consecutive days | Repo pushed to GitHub; Apify token and Telegram bot token as repository secrets | not-started |
| 1b | Actor intelligence bank: first pull and first synthesis (runs alongside 1) | `bank/snapshots/<date>.json` from the Store API (top 300 by users plus top 20 per data-shaped category); one page per actor under `bank/actors/`; first `bank/patterns/successful.md` and `unsuccessful.md` with every claim linked to actor pages; first `bank/gaps.md`; the weekly pull on GitHub Actions cron and the monthly synthesis on a Claude cloud routine (manual `/bank-refresh` as fallback) each proven by one real run | Read-only Apify token as a repository secret; repo on GitHub | not-started |
| 2 | Actor #1: picked from `bank/gaps.md` after a Store gap check | Fixture proposed by the builder and approved by the user; builder PR passed adversarial review; ruling in `decisions.md`; published open source, pay-per-event, limited permissions, priced under the incumbent; maintainer green on publish day | 1, 1b | not-started |
| 3 | Actor #2, same route | As milestone 2 | 2 | not-started |
| 4 | 30 unattended days | Maintainer log shows 30 consecutive daily runs across both actors with at least one repair or named escalation | 3 | not-started |
| 5 | Grok Bot watcher (deferred) | A daily routine reads Tranco deltas, Product Hunt / Hacker News launches and X, and hands over a ranked shortlist; no credentials for this repo on the Grok Bot computer | 2 and 3 earning | not-started |

## Risks & unknowns
| Risk | Known / Unknown | Mitigation, or cheapest test to resolve it |
|---|---|---|
| Chosen site sits behind anti-bot (Cloudflare, DataDome) and needs residential proxies, so the actor runs at a loss | Unknown | Plain `fetch` of three target pages before committing to a site; price from measured compute units on the fixture run |
| In-app scheduled tasks fire only while the app is open, so the maintainer cannot live there | Known | Ruled 2026-10-18: GitHub Actions cron; milestone 1 proves 7 unattended days before any actor depends on it |
| Public professional data is still personal data under UK GDPR; a complaint or takedown lands on the user's KYC'd account | Known | Lawful basis recorded in each ruling request; private and sensitive data excluded; honour takedown requests by pausing the actor |
| The Apify token and Telegram bot token live as GitHub repository secrets; a workflow change could exfiltrate them | Known | Secrets scoped to one workflow; PRs that touch `.github/` get the same adversarial review as actor code; no third-party actions without a pinned SHA |
| Apify account has no KYC, so no payout and no agentic-payment eligibility until it is done | Known | KYC is a human step; the plan assumes it happens before milestone 2's publish, which the user has not yet confirmed |
| Pay-per-event pricing below platform cost (negative profit) | Known | Weekly read of profit per actor in Apify Actor analytics; `apify-actor-start` synthetic event on; memory limits set |
| Nobody finds the actors on the Store | Unknown | Read paid and free user counts 30 days after each publish; promote per Apify's marketing checklist only if zero |
| Grok Bot shares one computer and its credentials across every bot on the account | Known | The watcher is read-only; the Apify token lives in this repo's `.env` and is never placed on the Grok Bot computer |
| A target site's terms forbid scraping, or the data turns out personal | Known | Human ruling before build, recorded in `decisions.md`; hard exclusion in the brief |
| A Claude cloud routine cannot reach the repo or run the synthesis on the plan | Unknown | Milestone 1b's first synthesis run; if it fails, the `/bank-refresh` skill run by hand is the fallback and the plan row says so |
| The bank costs more attention than it saves (Kun: durable loops pay off only with recurring inputs) | Known | Overridden at interview; judged by whether `bank/gaps.md` produces actor #1 and whether 1b's hours come out of milestones 1–3 |
| The builder cannot handle a site unaided and the user finishes it by hand | Known | Expected for roughly half of early candidates; each fix lands in the actor template so the next build absorbs it |

## Resourcing
- Appetite (from feasibility §B4): multi-week — two actors plus the maintainer loop, about 4–6 weeks
- Time budget: roughly 6–8 hrs/week around a fixed 20 hrs/week of lectures, through December 2026 (free blocks and November/December collisions not given at interview; Signal's feature freeze 2026-11-22 and data freeze 2026-11-25 are known)
- Skill gaps + how closed: Apify SDK and Crawlee for Python (read the docs this session; closed by building the stub in milestone 1); anti-bot handling (closed per site, or the site is dropped)
- Costs: one-time — none; recurring — none new (existing Claude, Cursor/Grok Bot and Apify plans; the Creator plan's $500 prepaid usage covers testing for 6 months); per-use appetite (§B4b): existing plans only, no new recurring spend until an actor earns; visibility wanted — profit per actor from Apify analytics weekly, compute units per maintainer run in its log
<!-- PLAN:BODY:END -->
