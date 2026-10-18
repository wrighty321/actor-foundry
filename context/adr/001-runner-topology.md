# 001 — Runner topology: a deterministic daily check, a separate manual repair workflow
Status: accepted — 2026-10-18, by the user

## Context
- The maintainer must run every day whether or not this machine is on. In-app scheduled tasks
  fire only while the Claude app is open (vault `knowledge/scheduled-agents-run-locally.md`,
  measured 2026-07-28), so they are not a runner. GitHub Actions cron was ruled 2026-10-18.
- The repo is private (`gh repo view` 2026-10-18), so GitHub's 60-day auto-disable of scheduled
  workflows on inactive public repos does not apply. Actions minutes on a private repo are
  metered — **assumed** to stay inside the free allowance at one short job per actor per day.
- Only two steps need a model: proposing a repair and naming a root cause. Checking a fixture,
  pausing an actor and sending an escalation are deterministic.
- The model credential has the widest blast radius of the secrets. Unattended Claude Code on a
  subscription (`claude setup-token`) is documented for CI but limited to "ordinary, individual"
  use (vault `knowledge/claude-subscription-logins-serve-only-anthropics-own-tools.md`, read
  2026-10-01); heavy headless use is a terms risk, not a settled yes. An API key is a separate
  prepaid pot and new recurring spend (vault `knowledge/anthropic-api-credits-are-not-subscription-limits.md`).
- Pinned model IDs fail silently on retirement (vault `knowledge/pinned-model-ids-break-silently-on-retirement.md`).
- A pause that depends on a model call is a pause that fails when the model is down.

## Decision
Two GitHub Actions workflows with disjoint secrets:

1. `maintainer-check.yml` — cron, once per actor per day. Deterministic Python
   (`uv run foundry check <actor>`): runs the fixture's two layers (ADR 002), appends one log
   line (ADR 005), and on failure sets the kill switch and flagged description (ADR 003) and
   sends a Telegram escalation whose "named cause" is structural and deterministic — which
   anchors failed, row-count delta, HTTP status, which fixture page. Secrets: `APIFY_TOKEN`,
   `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`. No model.
2. `maintainer-repair.yml` — `workflow_dispatch` only, started by the user from the Telegram
   link (or, later, by the check for a failure class that has climbed the trust ladder). Runs
   the Claude Code CLI with `CLAUDE_CODE_OAUTH_TOKEN`, no pinned model ID, bounded by
   `--max-turns` and a job timeout, with the failing fixture output and the actor source as its
   brief. It opens a pull request and stops. It never runs `apify push`.

Deployment is a third workflow on merge to `main` (ADR 004). The check workflow is the only
one with a schedule.

## Alternatives
- **One workflow with the model in the daily loop.** Rejected: the widest-blast-radius secret
  runs every day, a model outage or quota wall blocks the pause, and subscription use becomes
  the daily heavy pattern the terms warn about.
- **An always-on machine, or Grok Bot.** Rejected: Grok Bot shares files and credentials
  across every bot on the account (vision, 2026-10-18); another machine is another thing to
  keep alive before any actor earns.
- **An Apify Actor on an Apify Schedule as the checker.** Rejected: it cannot commit the log
  or open pull requests, it still needs the Apify token, and it splits one loop across two
  schedulers.
- **Local Claude scheduled tasks.** Rejected on the vault's measurement: they fire on next
  app launch, not at the slot.

## Consequences
- Easier: pause and escalation never wait on a model; each secret sits in exactly one
  workflow; failure classes can be counted from deterministic diffs, which is what the trust
  ladder needs.
- Harder: the daily "named cause" is structural (what broke), not causal (why); the why
  arrives only when the repair workflow runs. Accepted — the brief asks for a named cause on
  escalation, and "anchor `price` missing on page 2 of 3" is one.
- Commits us to: a `foundry` CLI with `check`, `pause`, `unpause`, `pull`, `summary`
  subcommands (ADR 006); Actions as the single scheduler; watching the OAuth-in-CI terms (an
  API-key route needs a ruling and a cost line); the repair workflow stays rare and manual
  until a failure class earns automation.
