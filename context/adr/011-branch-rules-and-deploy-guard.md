# 011 — Branch rules and the deploy guard: only the user's push deploys
Status: accepted — 2026-10-18, by the user (proposed the same day by the Kun pass on the architecture)
Supersedes: the "no required-PR rule" consequence of 004; the "commit to `main`" clause of 007

## Context
- `deploy.yml` is the only caller of `apify push` and fires on any push to `main` under
  `actors/**`.
- The cloud routines (ADR 007) read the open web and hold a standing repo-write credential;
  their path restriction was a sentence in a prompt file, not a control. The repair job (when
  it exists) runs a model with target-site HTML in its brief. One misread or injected
  instruction that lands a commit under `actors/x/src/` on `main` would deploy to the Store;
  L1 cannot catch a change that keeps rows identical.
- ADR 004 kept `main` unprotected because it believed the Actions bot could not be exempted
  from a required-PR rule. GitHub rulesets allow a bypass list that includes GitHub Apps, with
  a "for pull requests only" mode (docs.github.com, read by the independent review
  2026-10-18 — documented, not yet exercised on this repo).
- A push produced by merging in the GitHub UI carries `github.actor` = the merging user
  (documented, same read).
- Vault `knowledge/incident-response.md`: a shared store two agents can both write becomes a
  covert channel.

## Decision
1. **A ruleset on `main` requires pull requests**, with the GitHub Actions app on the bypass
   list so the daily check's log and status commits land directly (ADR 005, 013).
2. **The deploy job runs only when the push actor is the user:**
   `if: github.actor == 'wrighty321'`. A commit under `actors/**` that reaches `main` any other
   way deploys nothing.
3. **Agents never commit to `main`.** Cloud routines open pull requests (candidate notes,
   synthesis pages, playbook changes); the repair job opens pull requests. The user merges.
   For candidate notes the Telegram line reads "merge to land, close to discard" — the same
   grammar as repairs.
4. **Every agent-made commit carries a trailer** — `Agent: <routine or job name>` and
   `Run: <url or id>` — so `git log --format='%(trailers)'` separates the user's writes from
   agents'.

## Alternatives
- **The deploy guard alone, `main` unprotected.** Rejected as the sole control: it closes the
  Store path but still lets agent-written text land on `main` unreviewed; the ruleset is one
  setting.
- **Path-restricted push rules** (only `actors/**`, `.github/**`, `wiki/playbook/**` protected).
  Deferred: whether this repo's plan offers push rulesets with file-path restriction is
  **assumed**; if it does, it replaces the Actions bypass. Checked when the ruleset is created.
- **A separate `research` branch for the routines.** Rejected: Obsidian reads `main`; a second
  branch is a second place to look.
- **Branch protection (classic) instead of a ruleset.** Rejected: no app-level bypass.

## Consequences
- One more tap per candidate batch (merge the PR) before the note appears in Obsidian — and a
  glance at it on the phone first.
- Log and status commits need the bypass; if the bypass does not take effect, the check's
  commits fail visibly and the fallback is a `maintainer-log` branch (a new ADR, not a quiet
  change).
- The routines' prompt files say "open a pull request", and the review session treats
  `routines/*.md` like `.github/`.
