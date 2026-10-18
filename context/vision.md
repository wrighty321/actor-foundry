# Vision — actor-foundry

Why this route, what was rejected, and what would prove it wrong. Read before any directional
change. Written 2026-10-18 from the session that created the repo; the brief and plan hold the
frozen intent and the living state, this holds the reasoning between them.

## Decision provenance
- **2026-10-18, `random` session (transcript id `ff38ae06` / `010e448b`).** The user asked for
  automatable online ventures; Apify actors ranked first for fit. Apify's monetisation docs were
  read the same day (`docs.apify.com/actors/publishing/monetize`, pricing, payouts, pay-per-event,
  Standby, permissions): 20% commission, platform costs deducted from pay-per-event payouts,
  $20 PayPal/Wise payout minimum, KYC required, agentic payments need pay-per-event plus limited
  permissions.
- **The user's proposal:** three agents — a research scout, a growth watcher that ships a
  scraper early when a site's growth rate looks promising, and a scraper builder.
- **Claude's review the same day:** demand should drive selection with growth as a modifier; a
  human ruling belongs before any build; a maintainer agent was missing and is the part that
  decides whether the portfolio compounds or rots; prove the loop on two hand-picked sites
  before adding the watcher.
- **Grok Bot comparison, same day,** from xAI's docs (`docs.x.ai/grok-bot/overview`, page dated
  2026-10-07): per-bot charters yes, isolation no — one shared computer per account with shared
  files, browser sessions and CLI credentials; weekly usage allowance. Ruling: hybrid — Grok Bot
  for the read-only watcher, Claude Code for builder and maintainer, the user's ruling between.
- **Kun Chen pass, same day,** against `kunchenguid/kun` at commit `6182bc1` (his `OPINIONS.md`
  and about seventeen X posts, 2026-08-09 to 2026-09-30). Applied, by the user's ruling: the
  human picks targets with a Store gap check; the maintainer loop is built first because a
  failing fixture is a machine-verifiable objective and a new-site build is not; the user owns
  each actor's fixture rather than letting the builder write its own tests; adversarial review on
  builder PRs; the watcher is built last.
- **Gate answers, 2026-10-18:** name `actor-foundry`; telos trace none under R2; spend on
  existing plans only; kill criterion "no paid run by 2027-01-31".

## Rejected routes & named biases
- **Growth-first target selection** (the original watcher-as-funnel). Rejected: a rising site is
  a signal about the site, not about buyers; Kun's map — "good ideas start with named people who
  care about a real problem, not with technology-first excitement". Growth is kept as a timing
  modifier, and as the one non-commodity edge (see falsification).
- **Fully automatic shipping** with no human ruling. Rejected: publishing under a KYC'd developer
  account is an external send (telos R7); an unread site's terms and data class are judgement
  calls; Kun — "anything that has a real world consequence is very hard to fully decouple from
  human judgment".
- **Everything on Grok Bot.** Rejected for the builder and maintainer: shared computer and
  credentials across all bots; outside the apparatus (no `/consolidate`, no signals, no hooks);
  weekly allowance versus a daily maintainer. Kept for the watcher: native X reads, always-on,
  "setup is a message".
- **Everything on Claude.** Not rejected, deferred: the watcher's X feed is the one input Claude
  lacks natively; if Grok Bot's isolation or allowance turns out worse than documented, the
  watcher moves to Claude and loses X.
- **Builder-written tests.** Rejected on Kun's DeepSWE finding (agent-written tests are a second
  copy of the agent's reading of intent, so a failure cannot say whether code or test is wrong).
  The fixture is human-owned.
- **Named biases:** the reviewer of this plan (Claude) is not neutral on the Claude-versus-Grok
  question, and said so; Kun's public map is distilled from posts, is a subset of his judgement,
  and never mentions Apify.

## Operator overrides
- **Building despite a telos trace of "none".** The feasibility gate records an untraced venture
  as a signal against building. Overruled because the running cost is bounded to existing plans
  and the maintainer loop is worth proving for P12 even at zero revenue. Judged by: the kill
  criterion, and whether the maintainer's 30 unattended days (milestone 4) actually happen.
- **Keeping the watcher at all.** Kun's map would drop it until adoption justifies a durable
  loop; the user kept it as milestone 5, gated on two actors earning. Judged by: whether the
  shortlist it produces ever becomes a shipped actor that pays.

## Dates & tripwires
- **2027-01-31** — kill criterion: no paid run by a stranger, abandon or pivot (frozen in
  `commitments.md`).
- **Slip rule:** milestone 1 (maintainer loop, 7 unattended days) not done by **2026-11-15**
  with no named blocker ⇒ the runner choice is wrong, not the loop; switch runner before writing
  any actor code. Milestone 3 (second actor) not published by **2026-12-31** with no named
  blocker ⇒ the build cost was misjudged; milestone 5 is struck and the plan shrinks to keeping
  two actors alive.

## Falsification guardrails
- **"A machine can keep a niche actor green."** Falsified if, over milestone 4's 30 days, the
  user spends more than 2 hours a week on repairs the maintainer should have made. Then: the
  thesis is wrong, the portfolio does not compound, and the kill decision is taken on the
  evidence even before 2027-01-31.
- **"Being early on a rising site is worth more than the copy that follows."** Falsified if the
  first actor shipped on a rising site is matched by a competing actor within 30 days and loses
  its paid users to it. Then: the watcher (milestone 5) is struck permanently and selection is
  gap-check only.
- **"Buyers exist for niche data."** Falsified by the kill criterion itself.

## Success gates
1. Milestone 1's loop has 7 consecutive daily log entries from an unattended runner.
2. Actor #1 is public, pay-per-event, limited permissions, with a user-owned fixture, a reviewed
   PR and a dated ruling.
3. Actor #2 the same.
4. 30 consecutive unattended maintainer days with one real repair or named escalation.
5. One paid run by a stranger (also the kill criterion's complement).

## Risk register
| Assumption | Validated by |
|---|---|
| A stable runner exists outside the in-app scheduler | Milestone 1 |
| Target sites can be fetched without residential proxies | Milestone 2, plain-fetch test before the site is chosen |
| Pay-per-event prices clear platform cost | Milestone 2 publish-day analytics; weekly thereafter |
| The Store surfaces new actors at all | Milestone 2 and 3, 30-day user counts |
| Grok Bot can run a read-only routine with no repo credentials | Milestone 5 |
| The builder handles roughly half of candidates unaided | Milestones 2 and 3, counted |

## Standing state at decision time
- No actor, fixture or template exists. The user has never built an Apify actor by hand; the
  first build is the walkthrough the brief could not record.
- Signal (`wrighty321/signal`), a scheduled LLM research-delivery pipeline in Python, is the
  nearest existing system and the reason the stack is Python.
- Grok Bot facts and the Apify platform facts are in `context/research-2026-10-18.md` (also
  mirrored as `random`'s memory `grok-bot-capabilities.md`); the Kun clone was read from a
  session scratchpad and is not kept.
- Telos (`vault/telos.md`, reviewed 2026-08-31): P12 is the only problem; the commercial chain
  X13 is retired; R2 governs this as tinkering; R7 keeps publishing human-approved.
