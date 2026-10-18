# 007 — Research-agent runners: deterministic pull on Actions, model steps as cloud routines, notifications from Actions
Status: accepted — 2026-10-18, by the user
Amended 2026-10-18: notifications, target cadence and the `store-gaps` writer by 013; direct commits to `main` by 011 (routines open PRs); the consequence "testable offline against a saved Store page" is corrected in architecture.md §Snapshot — the live list's envelope `count` overstates the items returned.

## Context
- The weekly Store pull needs no model and no secret (every field verified public on
  2026-10-18; `wiki/README.md`). The monthly synthesis and the weekly target research need a
  model.
- In-app scheduled tasks fire only while the app is open and carry no `CLAUDE.md`, no
  `CLAUDE.local.md` and no memory (vault `knowledge/scheduled-agents-run-locally.md`,
  `knowledge/subagents-and-routines-start-memory-blind.md`). A cloud routine has the same
  blindness: every convention it depends on must be in a file it is told to read.
- Ruled 2026-10-18: repository secrets live in GitHub Actions, nowhere else. A cloud routine
  that pings Telegram would hold a second copy of the bot token.
- Running the synthesis as Claude Code inside Actions would make subscription use the daily
  heavy pattern the terms warn about (vault, ADR 001).
- **Assumed:** a Claude cloud routine can clone this private repo and push a commit. Verified
  by milestone 1c's first run; the fallback is the `/wiki-refresh` skill run by hand.

## Decision
- **Weekly pull** — `wiki-pull.yml`, cron, `uv run foundry pull`: pages the public Store list,
  ranks locally by `totalUsers30Days`, takes the top 300, the top 20 per data-shaped category
  and the weak-actor sample, fetches `/acts/{id}` and `/builds/default` for each, writes
  `wiki/evidence/snapshots/<date>.json` and regenerates `wiki/evidence/actors/*.md`, commits
  under `wiki/evidence/**`. No secrets. Deterministic.
- **Monthly synthesis** (milestone 1d) and **weekly target research** (milestone 1c) — Claude
  cloud routines. Each reads one self-contained prompt file first — `routines/wiki-synthesis.md`,
  `routines/target-research.md` — which restates every convention the routine needs (evidence
  grades, candidate card, exclusions, what it may and may not write). They commit to `main`
  under `wiki/synthesis/**` and `wiki/candidates/**` only; a change to `wiki/playbook/**` is a
  pull request, never a commit. They hold no secrets.
- **Notifications** — `notify.yml` in Actions, triggered by pushes touching
  `wiki/candidates/**` (new or changed candidate notes → one Telegram message listing slug,
  pitch and score) and by the maintainer (ADR 001). The bot token stays in Actions.
- **Fallback** — `/wiki-refresh`, a repo skill that runs the synthesis prompt by hand in an
  interactive session.

## Alternatives
- **Claude Code in GitHub Actions for synthesis and target research.** Rejected: puts the
  OAuth token in two more workflows and turns subscription use into a scheduled heavy pattern.
- **Local scheduled tasks.** Rejected on the vault's measurement.
- **Grok Bot routines.** Rejected for anything that writes this repo (shared credentials);
  kept only as a possible read-only X source later (milestone 5).
- **Routine pings Telegram directly.** Rejected: second copy of a secret outside Actions.

## Consequences
- Easier: the pull is testable offline against a saved Store page; the routines' prompts are
  reviewable files in the repo; secrets stay in one place.
- Harder: two runner kinds (Actions and cloud routines); the routine's repo access is
  unverified until it runs; a routine that misreads its prompt file silently writes wrong
  pages — which is why `wiki/playbook/` is PR-only and why the verification posture plants
  known flaws (architecture.md).
- Commits us to: `routines/*.md` as the single source of each routine's instructions, path
  rules for what each writer may touch, and `notify.yml` as the only Telegram sender outside
  the maintainer.
