# Build Plan — actor-foundry
brief: context/project-brief.md · spec: none (not a web app; generic lane) ·
handoff: context/architecture.md + context/adr/001–016 · approved: pending · revision: 1 (drafted 2026-10-19)

Build target: a Python monorepo whose `foundry` package runs a deterministic daily maintainer
over fixture-defined actors on Apify — preflight, charged push, pause, status, Telegram — with
deployment and research as OS-shaped workflows gated by the user's merges; built on a private
stub first, then actor #1 and #2 through the wiki's candidate pipeline. The build runs in a
separate session (user ruling, 2026-10-18); this file is what that session works.

## Protocol (any session working this file)
Execute the next unblocked step. Run `verify-by` exactly as written; record evidence inside
`status: done (evidence: what ran → what came back)`; a failed verify stops execution — fix
within the step's scope or mark `blocked (<reason>)` and go to the gate. At every stage gate:
run the accumulated suite (`uv run pytest` at the workspace root, every test earlier stages
committed), review the stage's diff (`/code-review` medium scoped to the stage), re-read the
frozen brief for drift, log faults as `/signal` records, write `gate: PASSED <date> | OPEN |
blocked (<reason>)`, then stop for the user's go. Amendments are appended to `

# Completed upgrade stages (carried forward verbatim; never edited)

## Stage U1 — Upgrades 2026-10-18 (independent assessment with Kun Chen's opinion map as lens)

Source: `/upgrade` run of 2026-10-18 against the whole repo, operator-prioritised, job statement
confirmed as "grow a portfolio of paid data products faster than one person can keep them
working by hand". Items accepted by the user in advance ("those proposals … should be
integrated"). The evaluator's deferred list is at the end of this stage.

### U1.1 — Close the hole in the `.env` guard (A1 · observed)
- accepts: `Read` of `.env.example` succeeds; `Read` of `.env` / `.env.local` is denied; a
  `Bash` command whose text references `.env` (not `.env.example` / `.env.sample`) is denied.
- verify-by: in a session, `Read .env.example` (expect success) then a Bash command
  containing `cat .env` (expect the PreToolUse hook to deny it).
- evidence: 2026-10-18, this session, after editing `.claude/settings.json` — `Read
  .env.example` returned the file (it was blocked before the edit); a Bash heredoc whose text
  contained `.env` was denied with `PreToolUse:Bash hook error`. Caveat recorded in
  `.env.example`: the guard is a name heuristic and misses indirect reads.
- status: done

### U1.2 — Settle the maintainer's three open definitions before any cron (A2 · inspected)
- accepts: milestone 1's "Done when" names the fixture comparison rule, the pause call and the
  approval transport, and requires a hand-run week on a private stub to settle each before the
  cron is written; the pause design does not depend on the Store `notice` field.
- verify-by: read `context/project-plan.md` milestone 1 and the two new risk rows; read
  `CLAUDE.md`'s maintainer lines; confirm `notice` is absent from the Update Actor request
  body in the Apify client references.
- evidence: plan row 1 rewritten; risk rows added for the `notice` finding and the track order;
  `CLAUDE.md` names the kill-switch record plus flagged `description`. `notice` absent from
  `UpdateActorRequest` (Python client reference) and from `ActorClient.update`'s `Pick<…>`
  (JS client reference), and present only in the response schema of the fetched
  `api/v2/act-put` page — three reads, 2026-10-18. The design itself (comparison layers,
  failed-vs-zero-rows exit on pause, approval grammar) is handed to `/architect` as ADR input.
- status: done here; the stub week is **elsewhere** (milestone 1)

### U1.3 — Put the runner's Claude credential in the inventory and the cost line (A3 · inspected)
- accepts: `.env.example` names `CLAUDE_CODE_OAUTH_TOKEN` and the workflow it is scoped to;
  `CLAUDE.md` and the plan's exfiltration risk row cover it; the cost line states the route.
- verify-by: grep `CLAUDE_CODE_OAUTH_TOKEN` across `.env.example`, `CLAUDE.md`,
  `context/project-plan.md` — three hits minimum.
- evidence: see the integration check below.
- status: done

### U1.4 — Make the wiki's data contracts match the Store API (A4 · observed)
- accepts: `wiki/README.md` names the verified public fields, the five run statuses, the
  agentic-payments flag, the permission level, the tier basis for prices and the lack of a
  server-side 30-day sort; the actor-page stats table has the five statuses; `candidate-card.md`
  names the price tier and the fetch IP class; `rulings-log.md` carries four sub-scores.
- verify-by: `curl -s "https://api.apify.com/v2/store?limit=1"` and check the keys named in
  the README are present; read the three files.
- evidence: 2026-10-18 unauthenticated calls — `/v2/store` item carried `stats.totalUsers30Days`,
  `publicActorRunStats30Days` = {ABORTED, FAILED, SUCCEEDED, TIMED-OUT, TOTAL},
  `isWhiteListedForAgenticPayments`, `currentPricingInfo.pricingPerEvent`; `/v2/acts/{id}`
  carried `actorPermissionLevel`, `readmeSummary`; `/v2/acts/{id}/builds/default` carried
  `readme` and `inputSchema`; total listed actors 91,483. Files edited as named.
- status: done (the commit of `wiki/` to `origin/main` is pending the user's go)

### U1.5 — Remove the read-only Apify token (S1 · observed)
- accepts: `APIFY_TOKEN_READONLY` appears nowhere in the repo; milestone 1b's "Blocked on" is
  empty.
- verify-by: `grep -rn APIFY_TOKEN_READONLY .` returns nothing; read plan row 1b.
- evidence: see the integration check below.
- status: done

### U1.6 — Shrink milestone 1b; move the synthesis routine to 1d (S2 · inspected)
- accepts: 1b is done on one committed snapshot, one page per actor and a first graded
  `store-gaps.md` entry; 1d holds the synthesis routine, the `/wiki-refresh` skill and the first
  playbook PR, blocked on the second snapshot; `wiki/README.md` says the first synthesis waits
  for two snapshots.
- verify-by: read plan rows 1b, 1c, 1d and the README's synthesis section.
- evidence: rows rewritten; README section added; decisions line appended.
- status: done

### U1.7 — Park the freshness-receipt bet behind its check (Track B · estimated)
- accepts: the plan carries the bet as a parked row with its validation check and kill
  condition; nothing is built.
- verify-by: read plan row B1.
- evidence: row added; decisions line appended.
- status: done

### Integration check
`grep -rn "APIFY_TOKEN_READONLY\|bank/" --include=*.md --include=*.example .` returns only
historical lines in `context/decisions.md` and the dated `context/research-2026-10-18.md`;
`grep -rln CLAUDE_CODE_OAUTH_TOKEN .` lists `.env.example`, `CLAUDE.md`,
`context/project-plan.md`; `node -e "JSON.parse(require('fs').readFileSync('.claude/settings.json','utf8'))"`
parses; every `[[wikilink]]` in `wiki/` resolves. Result recorded below.

Result, 2026-10-18: the only `APIFY_TOKEN_READONLY` / `bank/` hits outside the two historical
files are this stage's own verify text and the wiki README's rename sentence; the OAuth token
name is in `.env.example`, `CLAUDE.md`, `project-plan.md` (plus `decisions.md` and this file);
`settings.json` parses; 15 wiki pages, every link resolves (the one "unresolved" hit is the
literal word `[[wikilinks]]` inside a code span in the README, not a link). Passed.

### Validation hooks (what confirms these worked in the world)
- U1.1: a fresh session repeats the two tool calls in `verify-by`.
- U1.2: on the private stub, write the kill-switch record and the flagged description through
  the API, `GET` both back, break the parser, watch the fixture fail, then restore. Ten minutes.
  If the kill switch cannot be read before charging starts, the pause design changes before any
  cron is written.
- B1 (bet): one question in r/webscraping or the Apify Discord — "choosing between two actors
  for the same site, what do you check?" — plus one `search-actors` read of what an agent is
  shown for a data type with 3+ competitors. No "does it work today" in the answers kills it.

### Deferred (from the assessment; parking reasons stated)
- Define "match" for escalation classes on the trust ladder — nothing to measure until the
  first escalation; the rulings log covers candidates only.
- `wiki/synthesis/topics/<category>.md` — no approved candidate names a category yet.
- `wiki/first-party/` page format — no live actor until milestone 2.
- Unattended profit-per-actor in the weekly summary — no analytics API found (two docs
  searches 2026-10-18; a reading, not a state); the one manual weekly step until one is found.
- Grok Bot X feed, Standby/MCP endpoint, second Apify account — already deferred; no buyer
  or event has asked.
- Non-English sites — no demand evidence; revisit only if a candidate's signals are non-English.
- Weekly summary as HTML — Telegram text suffices at two actors.
- `.gitattributes` with `* text=auto eol=lf` — one line; every file warns LF→CRLF today and will
  churn diffs between Windows and the Actions runner. Do with the first `.github/` PR.
- Split `CLAUDE.md` into stable conventions and dated rulings — cosmetic at this stage.
- Terminology sweep `bank` → `wiki` in `research-2026-10-18.md` §2 and `vision.md`'s override
  paragraph — historical text; left as written.
- Discarded: buyer-supplied fixtures as the order; the maintainer loop as a service to other
  developers — both cross the brief's boundary condition.

## Stage U2 — Kun pass on the architecture, 2026-10-18

Source: an independent review of `context/architecture.md` and ADRs 001–008 with Kun Chen's
opinion map as the lens, 2026-10-18. Five Must and twelve Should proposals; integrated at the
user's instruction. Reversals of accepted decisions are new ADRs (009–013, `proposed`); the
accepted ADRs carry a status amendment line and are otherwise untouched. Planning items are
done here; code items are **elsewhere** and listed with their `verify-by` for the build session.

### U2.1 — In-actor preflight and page-buffered charging (M1 · inspected)
- accepts: ADR 009 exists; `fixture.md` gains `min_complete_share`; B22 states the run order;
  exit code 12 is reserved; the check counts exit-12 runs.
- verify-by (elsewhere): a fake `Actor` test shows zero `push_data` calls when the preflight
  fails or a page is below `min_complete_share`; on the stub, a deliberately broken live page
  ends a run with exit 12 and zero charges.
- evidence: ADR 009, architecture.md C2 and failure rows, B22 written.
- status: done here; test and stub run elsewhere

### U2.2 — Repair, fixture re-approval, un-pause gate (M2 · observed)
- accepts: ADR 010 exists; the `repair_id` gate is gone; deploy un-pauses only on
  `status: approved` with `approved_on` on or after the repair commit; B3 names the draft rule.
- verify-by (elsewhere): a merged repair with a `draft` fixture leaves the actor paused and
  sends exactly one "fixture needs your approval" line.
- evidence: contradiction traced — ADR 003's record never gained a `repair_id`, ADR 001 gave
  the repair job no Apify token, and ADR 002's L1 cannot pass after a real site change without
  a fixture change only the user may make. ADR 010, B3, architecture.md updated.
- status: done here; deploy behaviour elsewhere

### U2.3 — Branch rules, deploy guard, agent PRs and trailers (M3, S10 · observed)
- accepts: ADR 011 exists; the deploy job runs only when the push actor is the user; routines
  and the repair job open PRs; agent commits carry `Agent:` and `Run:` trailers; a ruleset
  requires PRs on `main` with the Actions app bypassed.
- verify-by (elsewhere, operator step): the repository's rulesets list the rule; a test commit
  under `actors/**` pushed by the Actions bot produces no deploy run; a log commit from the
  check lands.
- evidence: ADR 011; architecture.md triggers and C10/C11 rows; wiki README; CLAUDE.md bullet.
  GitHub docs on ruleset bypass lists and the push actor on UI merges were read by the reviewer
  on 2026-10-18 (documented, not yet exercised on this repo).
- status: done here; ruleset creation is the user's step

### U2.4 — Store pull reconciliation (M4 · observed)
- accepts: the snapshot header carries `totals_seen`, `pages`, `items_received`,
  `unique_ids`, `previous_unique_ids`; the pull reconciles on unique ids, never on the
  envelope; a partial pull writes nothing.
- verify-by (elsewhere): a committed Store page with `count` greater than its items makes the
  old rule pass and the new rule fail; the first real pull's header numbers add up.
- evidence: reproduced here 2026-10-18 — `GET /v2/store?limit=1000&offset=0` twice: envelope
  `total 90920, count 1000`, **948 items** each time, identical id sets; `limit=100&offset=500`:
  `count 100`, **93 items**. architecture.md §Snapshot, wiki README, plan row 1b and a risk
  row updated; ADR 007's status note corrects its offline-testability consequence.
- status: done here; test elsewhere

### U2.5 — One secrets inventory; the repair PR carries its own L1/L2 (M5 · observed)
- accepts: `.env.example` is the single inventory and names each workflow's set; CLAUDE.md
  and the ADRs point at it; the repair job (when built) writes L1/L2 and the run URL into the
  PR body and sends the Telegram line itself.
- verify-by: a grep for the OAuth token name shows one inventory and pointers, no second list.
- evidence: three files disagreed on C5's secrets (ADR 001, the secrets example, architecture.md);
  bot-opened PRs do not get automatic `pull_request` runs (GitHub docs, reviewer's read
  2026-10-18). Files rewritten.
- status: done

### U2.6 — Status page, loud exit, off-hour cron (S1 · inspected)
- accepts: ADR 013 §5; `maintainer/STATUS.md` shape in architecture.md; the check exits
  non-zero on anything awaiting a human; crons off the hour.
- verify-by (elsewhere): a deliberately broken fixture produces a GitHub failure e-mail the
  same morning; `STATUS.md`'s mtime equals the last run.
- status: done here

### U2.7 — Pause as kill switch only; scoped token for the check (S2 · documented, carried as assumed)
- accepts: ADR 012 supersedes 003; `APIFY_TOKEN_SCOPED` is in the inventory; `foundry flag` /
  `unflag` are manual with a restore guard.
- verify-by (elsewhere): the scoped token lists the stub's runs and writes the control store;
  the daily workflow holds no unscoped Apify token.
- evidence: the scoped-token capability is the reviewer's read of Apify's integrations docs
  (2026-10-18); the page failed to fetch twice from this session, so ADR 012 carries it as
  assumed until the token is created.
- status: done here; token elsewhere

### U2.8 — Notifications merged into the check (S3 · inspected)
- accepts: no separate notify workflow; C12 retired in architecture.md; the check sends
  candidate pings and the Monday summary.
- status: done

### U2.9 — Repair workflow deferred; local `foundry repair` (S4 · inspected)
- accepts: ADR 013 §1; ADR 001 status note; `foundry repair` in the CLI list; plan row 1 and
  the order row say so.
- status: done

### U2.10 — Target research on demand, cap two (S5 · inspected)
- accepts: ADR 013 §2; wiki README and plan row 1c say on demand until actor #1 is approved.
- status: done

### U2.11 — `store-gaps` table generated by the pull (S6 · observed)
- accepts: ADR 013 §3; architecture.md "Store-gaps table"; plan row 1b; wiki README.
- evidence: 1b required a graded entry that nothing in 1b could write (synthesis is 1d).
- status: done here; generator elsewhere

### U2.12 — L2 runs as an Apify run of the deployed build (S7 · inspected)
- accepts: architecture.md C4 row and ADR 002's status note say so; CU is read from that run.
- status: done

### U2.13 — Listing-change gate grammar (S8 · observed)
- accepts: the `decisions.md` line must carry the PR number; architecture.md "Deploy gate
  grammar"; ADR 004's status note; CLAUDE.md.
- evidence: "a line dated on or before the merge" was always true (the file already had 25).
- status: done

### U2.14 — Third-party text cannot reach the builder's instructions (S9 · inspected)
- accepts: B20 (named sections of the note are the brief), B21 (reasons from field names and
  codes only), `routines/*.md` reviewed like `.github/` (architecture.md hard constraints).
- status: done

### U2.15 — Control-before-charge as a committed test; test authority labelled (S11 · inspected)
- accepts: architecture.md's verification posture separates tests with the user's authority
  from implementer tests and names the call-order test.
- verify-by (elsewhere): the test exists and fails when the order is swapped.
- status: done here

### U2.16 — Price read-back on deploy (S12 · inspected)
- accepts: deploy reads `pricingInfos` and compares with the README's per-1k line; a mismatch
  is a Telegram line.
- status: done here; behaviour elsewhere

### Integration check
13 ADR files numbered 001–013 with no collisions; each has a `Status` line, four sections and
at least one rejected alternative with a reason; 003 reads `superseded by 012`; the amended
ADRs carry a status note and unchanged bodies. architecture.md references only ADRs that
exist; no separate notify workflow, `original_description` field or `repair_id` gate remains
outside the superseded ADR and this stage's text. Result recorded below.

Result, 2026-10-18: 13 ADR files 001–013, no duplicate numbers; every file has a `Status`
line, 4/4 sections and 2–6 rejected alternatives; 003 reads `superseded by 012`; the seven
amended ADRs carry a status note and unchanged bodies (their original `notify.yml` and
`repair_id` sentences remain, as append-only requires, and are the only such hits besides the
new ADRs that describe what they supersede). architecture.md references all 13 and no
missing file; 15 components in its table, C1 (library) and C12 (retired) deliberately absent
from the diagram; 15 wiki pages, every link resolves. Passed.

### Deferred (from the review; parking reasons)
- Retire C7 if every pause over milestone 4 came from a preflight the actor already enforces —
  decided on that evidence (would supersede ADR 012).
- Trim the empty `wiki/synthesis/` pages until the second snapshot — kept: they are the
  designed empty state and cost nothing; revisit at 1d.
- The freshness-receipt bet (B1) costs one KV write once the preflight exists — folded into
  ADR 009's consequences.
- Batch `push_data` per page — folded into ADR 009 (`push_rows`).
- `.gitattributes` eol — with the first `.github/` PR.
- Path-restricted push rulesets — checked when the ruleset is created; would replace the bypass.
- GitHub mobile support for `workflow_dispatch` and the "Approve workflows" banner — unverified;
  irrelevant while repair runs locally.

## Stage U3 — Quality-score pass, its Kun pass, and the fit with the personal OS, 2026-10-18

Source: the user's note *Apify Actor Quality Score Playbook* read against the repo; a draft
proposal (`context/quality-score-proposal.md`); an independent Kun-lens review of the draft and
repo; the user's instruction to follow the personal OS's agent and workflow templates. All
integrated at the user's instruction; ADRs 014–016 at `proposed`.

### U3.1 — Listing congruency and daily-test alignment (ADR 014 · observed/inspected)
- accepts: dataset schema generated from `fields`; prefill **and** default from the fixture;
  `foundry lint` with the structural rules and no word count; `listing.md` record; B14, B15,
  B23, B24 rewritten.
- verify-by (elsewhere): a lint fixture with one planted README/field mismatch fails; the
  stub's prefill run ends `Succeeded` with rows in under 2 minutes; `actor.json` title and
  description appear on the pushed actor.
- evidence: daily-test rule, eight categories and the dataset-view requirement confirmed in
  Apify docs this session; `prefill` is UI-only per the input-schema spec (reviewer's read).
- status: done here; code and stub elsewhere

### U3.2 — Operating for the score (ADR 015 · observed)
- accepts: pause classes (no pause for an L2 failure confined to the preflight page); `notice`
  and `isDeprecated` read from `/acts/{id}`; KYC as a publish gate; `publish-runbook.md`;
  Console Monday as the extended weekly step; never delete or deprecate; the repair fast path.
- verify-by (elsewhere): a forced preflight-only break on the stub produces `escalated
  (preflight guards)` and no pause; STATUS.md shows a `notice` column; the publish request
  template carries `kyc_passed_on`.
- evidence: Apify's 7-day self-heal rule and the KYC exclusion text (reviewer, 2026-10-18;
  totals 90,859 vs 91,585); no issues/reviews API (404s and no OpenAPI path, two shapes);
  `isDeprecated` in the Update Actor body.
- status: done here

### U3.3 — Fit with the personal OS (ADR 016 · inspected)
- accepts: `.os/project.yaml`, `.os/tools.yaml`, `.gitignore` for `.os/runs/`;
  `library/README.md`; `library/wf.foundry.target-research/` complete in the OS shape
  (workflow.yaml, template.yaml, lens.md, checks.yaml, four nineteen-field bodies, tests
  README with the planted set); three contracts under `library/contracts/`; `routines/*.md`
  retired before ever existing.
- verify-by (elsewhere): `foundry run-workflow wf.foundry.target-research --fixture
  tests/planted/` yields exactly one note (G1) and a PR body of one line; each contract schema
  validates its fixture document and rejects a planted bad one.
- evidence: the OS shape read from `workshop/context/os-plan/` (`06-wfag.md`, `04-system-plan.md`
  §wf./ag. and §pj., `06-build-programme.md` library layout) and `personal-os/library/schemas/`
  on 2026-10-18; the OS shell does not exist yet (personal-os first receipt targeted 2026-11-11).
- status: done here; sequencer and tests elsewhere

### U3.4 — Candidate card corrected (saturation) and extended (listing) (observed)
- accepts: saturation is a hand count of snapshot titles with Apify's bands, never the Store
  `search=` total; the unproven-band gate; `proposed_title`/`proposed_slug`; incumbent
  `modifiedAt`.
- evidence: `search=` totals this session 2026-10-18 — "google maps" 3,240, "linkedin jobs"
  2,023, "planning applications" 502.
- status: done

### U3.5 — Architecture, conventions, plan and wiki updated (observed)
- accepts: `architecture.md` v3 (C16, pause classes, lint, read-backs, OS runner note);
  `CLAUDE.md` bullets; plan rows 1c, 1d, 2 and five risk rows; wiki README (pull parameters,
  badge, id-diff deprecation, first-party schema, library workflows).
- status: done

### Integration check
16 ADR files 001–016, no duplicates; each has a `Status` line, four sections and rejected
alternatives; 003 superseded by 012; 009–013 amended by status note for 014–016; architecture.md
references all 16; every wiki link resolves; `.os/project.yaml` and the three contracts parse.
Result recorded below.

Result, 2026-10-19: 16 ADR files 001–016, no duplicates, every body with its four sections and
rejected alternatives; the handoff lists all 16; 16 wiki pages, every link resolves (including
the new `[[publish-runbook]]`); the three contracts parse as JSON; the four agent bodies carry
all nineteen fields at roughly 500–640 tokens each by chars ÷ 4 (a labelled estimate — the
build's `tests/test_bodies.py` measures with a tokenizer); YAML files checked by eye only (the
isolated interpreter has no yaml module). No `routines/` reference remains outside the ADRs
and this plan. Passed.

### Deferred (from the quality-score Kun pass)
- Tiered discounts — after the first paid run (five tiers exist).
- Task examples beyond one — after 14 clean daily tests.
- Rising Star — recorded as `badge` by the pull; never a target.
- `wf.foundry.build`, `wf.foundry.wiki-synthesis`, `wf.foundry.repair` directories — written at
  their milestones in the shape `wf.foundry.target-research` establishes.
- Co-author lines on planning-session commits — the user's ruling (ADR 016).
- `VISION.md` in the OS's sense — derived by the OS's project workflow later.

## Revisions`,
dated — never silent. Commit only when the user asks; a passed gate is the natural point to
offer. Every command uses `uv run`; bare `python` resolves to an unrelated venv on this machine.
Upgrade stages (`## Stage U*`) are completed, verified work and are never edited.

## Assumed premises carried as risks (settled by the step named; never built on as facts)
- `apify push` builds from the actor directory, so the `foundry` wheel must be vendored into it — S1.7.
- `.actor/actor.json` `title`/`description` ride `apify push` — S1.7.
- A scoped Apify token can list the actor's runs and write one named KV store — S1.7, S2.1.
- The owner's run list shows runs Apify's daily tester starts (`meta.origin == TEST`) — S1.7.
- `apify-default-dataset-item` and a custom `result` event would double-charge a push — S1.7.
- Apify's test runs are charged like any run — observed at S5.5, not relied on.
- GitHub rulesets let the Actions app bypass a required-PR rule; path-restricted push rules may or may not exist on this plan — S2.3.
- Actions minutes stay inside the free allowance — measured at S2.6.
- A cloud routine can open PRs through the interim sequencer — S4.3 runs locally first; cloud is a later reading.
- Obsidian Git accepts a base path one level above the vault — S3.4.

---

## Stage 1 — Walking skeleton: the stub week, hand-run (milestone 1's precondition)
The thinnest end-to-end path: a private stub actor on Apify that reads the control record,
preflights, extracts and charges per page; a fixture the user approved; `foundry check` run by
hand producing a log line, `STATUS.md` and a Telegram message; a forced break of each pause
class. Nothing scheduled, nothing public. Proves ADR 002, 003/012, 005, 006, 008, 009, 014 §1–2,
015 §1–2 and 016 §5 on the real platform before any cron exists.

### S1.1 — Workspace, package skeleton, test harness
needs:      —
scope:      `pyproject.toml` as a `uv` workspace (root package `foundry`, members `actors/*`); `ruff` config; `tests/` with a smoke test; `foundry/__init__.py`, `cli.py` with `--help`; `.os/project.yaml` commands resolve. Not: any Apify call.
accepts:    `uv run foundry --help` lists the subcommands named in architecture.md; `uv run ruff check .` is clean; the smoke test passes.
verify-by:  `uv run pytest tests/test_smoke.py -q` and `uv run ruff check . && uv run ruff format --check .`
status:     todo

### S1.2 — Contracts and bodies validated by tests
needs:      S1.1
scope:      `foundry/contracts.py` loads `library/contracts/*.schema.json` (jsonschema) and validates a document; `tests/test_contracts.py` validates one fixture document per contract and rejects one planted bad document each; `tests/test_bodies.py` checks every `library/**/agents/*.md` has the nineteen fields and measures its tokens with `tiktoken` (cl100k) against `budget`. Not: the sequencer.
accepts:    Three contracts validate their fixtures and reject the planted ones; four bodies ≤ 1,500 tokens measured, not estimated.
verify-by:  `uv run pytest tests/test_contracts.py tests/test_bodies.py -q`
status:     todo

### S1.3 — Control store client and `foundry init`
needs:      S1.1
scope:      `foundry/control.py`: read record (3 retries, backoff), write with read-back, the record schema of ADR 012, exit-code constants 10/11/12; `foundry init` creates `actor-foundry-control` with the unscoped token; `foundry pause|unpause` with read-back. Not: pause classes (S1.8).
accepts:    A fake KV test proves read-before-anything ordering, three retries then fail-closed, and that a write not read back raises; `foundry init` creates the store once on Apify.
verify-by:  `uv run pytest tests/test_control.py -q`; then `uv run foundry init` once (one-off, third-party) and paste the store id into this step's evidence.
status:     todo

### S1.4 — Fixture runner: L1, L2, preflight subset
needs:      S1.2
scope:      `foundry/fixture.py`: parse `fixture.md` frontmatter (validated as `ct.foundry.fixture@1`, written in this step), L1 (`extract(source/*) == expected.jsonl`, order-sensitive), L2 assertions (fields present and typed, `min_rows`, anchors exact|present|pattern|range), the preflight subset for `live_pages[0]`, and a structured failure list (assertion ids). Not: fetching.
accepts:    Against `actors/_template/fixture/`, L1 passes; a planted row change fails L1 naming the row; a planted anchor move fails L2 naming `company@1`.
verify-by:  `uv run pytest tests/test_fixture.py -q`
status:     todo

### S1.5 — Charged push per page
needs:      S1.3, S1.4
scope:      `foundry/charge.py`: `push_rows(rows, fields, min_complete_share)` wrapping `Actor.push_data(..., charged_event_name="result")`, complete-row share, `ChargeResult.event_charge_limit_reached` → persist and clean exit; a fake `Actor` recording call order. Not: the actor itself.
accepts:    Zero `push_data` calls when the preflight failed or the page's share is below threshold; the limit path exits cleanly with the count; the control read precedes any push in the recorded order.
verify-by:  `uv run pytest tests/test_charge.py -q`
status:     todo

### S1.6 — Actor template and the stub actor, local
needs:      S1.4, S1.5
scope:      `actors/_template/` and `actors/stub-books/` targeting `https://books.toscrape.com/` (a scraping sandbox: stable, no personal data, no terms against automation — chosen here because the handoff leaves the stub's site open; recorded as an assumption, swap if the user prefers another stable site): `src/main.py` (control read → preflight → fetch → `extract` → `push_rows` → `PERSIST_STATE`), `src/extract.py` pure, `.actor/actor.json`, input schema with `prefill` and `default` derived from the fixture, `.actor/dataset_schema.json` generated from `fields` by `foundry gen-dataset-schema`, README to the ADR 014 contract, `listing.md` (unpublished), `Dockerfile` installing the vendored wheel from `.vendor/` (gitignored). Fixture: builder drafts `source/`, `expected.jsonl`, `fixture.md` with `status: draft`; the user approves in their own commit. Not: Apify push.
accepts:    L1 green on the approved fixture; `apify run` locally yields `min_rows` complete rows; the generated dataset schema's fields equal `fields`; prefill/default equal `live_pages[0]`/`min_rows`.
verify-by:  `uv run pytest tests/test_fixture.py -q -k stub` and `uv run pytest tests/test_template.py -q` (dataset schema and prefill/default derived) ; then `cd actors/stub-books && apify run --purge` (one-off) and paste the dataset item count.
status:     todo

### S1.7 — Private push and the platform checks
needs:      S1.6
scope:      `apify login` launched as a detached OS process (human-paced; vault `interactive-logins-need-a-detached-process`); `apify push` of the stub, unlisted (no ruling needed); then, on the live stub: the vendored-wheel Docker build succeeds; `actor.json` title/description appear on `GET /v2/acts/{id}`; the prefill run (also Apify's future daily-test job) ends `Succeeded`, non-empty dataset, duration recorded (target < 2 min); kill switch set via `foundry pause` → a run exits 10 with the status message and zero `result` charges; `live_pages[0]` temporarily pointed at a wrong URL → a run exits 12 with zero charges; a scoped token (read actors + runs, write the control store) lists the runs and writes the record; `apify-default-dataset-item` removed and one `result` charge per row confirmed in the run's charge log; `publicActorRunStats30Days` excludes the owner's runs as documented. Not: anything public.
accepts:    Every bullet above has a run URL or API response pasted as evidence; the five assumed premises it covers are each marked settled or refuted in this file's risk list.
verify-by:  One-off, third-party (the posture names these as not holdable by tests): the checklist in this step with URLs and readings pasted under `status`.
status:     todo

### S1.8 — The check: L1, L2 as an Apify run, exit-12 count, `notice`, pause classes, log, STATUS, Telegram
needs:      S1.3, S1.4, S1.7
scope:      `foundry/check.py`: L1 offline; L2 as an Apify run of the deployed build with the prefill input, read back; exit-12 count from the run list since the last line; `notice`/`isDeprecated` from `GET /v2/acts/{id}`; pause classes of ADR 015 §1 (preflight-confined L2 failure → escalate only); fetch-error retry and second-day rule; log line (ADR 005 as amended) appended; `maintainer/STATUS.md` regenerated (ADR 013 §5 + `notice`); Telegram send-only client with the one-sentence clock; non-zero exit when a human is needed; `foundry summary` for Mondays. Not: the workflow file.
accepts:    Fakes prove each pause class routes correctly (five cases), the log line and STATUS.md render from a fixture log, a failed Telegram send still writes the line with `sent: false` and exits non-zero; by hand, `foundry check stub-books` appends one real line, regenerates STATUS.md and delivers a Telegram message.
verify-by:  `uv run pytest tests/test_check.py tests/test_status.py tests/test_telegram.py -q`; then `uv run foundry check stub-books` (one-off) with the Telegram message text pasted as evidence.
status:     todo

### S1.9 — Forced breaks, every class, hand-run
needs:      S1.8
scope:      Break the parser (planted regression in `extract.py`) → check logs L1 fail, pauses with read-back, escalates; restore, `foundry unpause`. Break only the live page's anchor → check logs `escalated (preflight guards)`, no pause; restore. Simulate fetch-error (unreachable `live_pages[0]`) → first day escalate without pause. Not: a repair PR.
accepts:    Three consecutive log lines show the three classes with the designed actions; the control record reads `paused: false` at the end.
verify-by:  `tail -n 4 maintainer/log/stub-books.md` pasted as evidence, plus `GET` of the control record.
status:     todo

gate: OPEN — suite (`uv run pytest`), stub-week measurements in S1.7, drift check against the brief, `/code-review` medium on the stage diff, user go.

## Stage 2 — Unattended: the maintainer on GitHub Actions (milestone 1)
Moves the skeleton onto the runner: secrets, the daily check, PR CI, the ruleset, the deploy
workflow with its guards and read-backs, then seven consecutive unattended days. Proves ADR 001,
004, 005, 010, 011, 013, 014 §3 and 015 §7 as workflows.

### S2.1 — Repository secrets and the scoped token
needs:      S1.7
scope:      Create `APIFY_TOKEN_SCOPED` (read actors and runs, write `actor-foundry-control`), set `APIFY_TOKEN`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` as repository secrets; `.env.example` unchanged or corrected to match. Not: `CLAUDE_CODE_OAUTH_TOKEN` (Stage 7).
accepts:    `gh secret list` shows exactly the four names; the scoped token's capability from S1.7 is recorded.
verify-by:  `gh secret list` (names only) pasted as evidence.
status:     todo

### S2.2 — `maintainer-check.yml`
needs:      S1.9, S2.1
scope:      Cron `23 6 * * *`; `uv run foundry check` per actor from a list in the workflow; commits under `maintainer/**` only with the Actions token; exits non-zero when the check does. Not: deploy.
accepts:    `workflow_dispatch` run is green, a log line and STATUS.md are committed by the Actions bot; a forced L1 break makes the run fail and GitHub e-mails the owner.
verify-by:  `gh workflow run maintainer-check.yml && gh run watch` (one-off) → paste the run URL, the commit SHA on `main`, and the failure e-mail's subject for the forced-break run.
status:     todo

### S2.3 — The ruleset on `main` with the Actions bypass (operator step)
needs:      S2.2
scope:      A ruleset requiring pull requests on `main`, GitHub Actions app on the bypass list ("for pull requests only" mode if offered); record whether path-restricted push rules exist on this plan. Not: branch protection (classic).
accepts:    `gh api repos/wrighty321/actor-foundry/rulesets` lists the rule; the next check-made commit still lands; a direct push to `main` from a non-bypass context is refused (exercise with a `git push` from a clone authenticated as the user — if the admin bypass makes it land, record that the rule was not exercised and why).
verify-by:  `gh api repos/wrighty321/actor-foundry/rulesets` pasted; the next `maintainer-check` run URL showing its commit landed.
status:     todo

### S2.4 — `deploy.yml` with guards and read-backs
needs:      S2.1, S1.6
scope:      Push to `main` under `actors/**`; job `if: github.actor == 'wrighty321'`; `uv run pytest`, `uv run foundry lint`, build and vendor the wheel, `apify push`, L1+L2 against the deployed build, read-backs (`pricingInfos` vs README and `listing.md`, `actorPermissionLevel`, `notice`, title/description), un-pause only when `fixture.md` is `approved` on or after the repair commit, `listing-change` gate requiring a `decisions.md` line with `#<PR>`; never sends `name`, `isPublic`, `isDeprecated`, `pricingInfos`, SEO fields; log line with merge SHA and merger. Not: publishing.
accepts:    A PR editing `actors/stub-books/src/extract.py` merged by the user → deploy green, log line with the SHA; a PR touching the README without a `#<PR>` line → deploy fails naming the missing line; a commit under `actors/**` reaching `main` via the Actions bot → no deploy run.
verify-by:  Three run URLs (green deploy, failed gate, skipped job) pasted; `uv run pytest tests/test_deploy_gate.py -q` for the gate grammar and read-back comparisons on fakes.
status:     todo

### S2.5 — PR CI
needs:      S1.1
scope:      `ci.yml` on every PR: `uv run pytest`, `uv run ruff check`, `uv run ruff format --check`, `uv run foundry lint`. Not: deploy.
accepts:    A PR with a planted lint violation (README H1) is red; the same PR fixed is green.
verify-by:  Two PR check URLs pasted.
status:     todo

### S2.6 — Seven consecutive unattended days
needs:      S2.2, S2.3, S2.4
scope:      No hand runs; `STATUS.md` shows consecutive log days; Actions minutes read from the billing page once. Not: any actor code change unless a break occurs (then S1.9's classes apply through the workflow).
accepts:    `maintainer/log/stub-books.md` has seven consecutive dates written by the Actions bot; minutes used are recorded against the allowance.
verify-by:  `grep -c '^2026-' maintainer/log/stub-books.md` and the seven dates pasted; the minutes reading pasted.
status:     todo

gate: OPEN — suite, stage diff review, drift check; milestone 1 done when S2.6 passes; user go.

## Stage 3 — Wiki pull (milestone 1b)
The weekly public pull with its reconciliation, the store-gaps table, its cron, and Obsidian
reading the result. Proves ADR 007 (as amended), 013 §3 and the snapshot contract.

### S3.1 — Store client and the pull
needs:      S1.2
scope:      `foundry/store.py` + `fn pull`: page `/v2/store` with `includeUnrunnableActors=true`, dedupe by id, reconcile `unique_ids` against `min(totals_seen)` with a stated tolerance and against the previous snapshot's count with a named maximum drop, record `query` parameters and `badge`, rank locally by `totalUsers30Days` (top 300, top 20 per category, weak sample), fetch `/acts/{id}` and `/builds/default` per sampled actor, find deprecated actors by id diff → `isDeprecated`, write `wiki/evidence/snapshots/<date>.json` (header as `ct.foundry.snapshot-header@1`, written in this step) and regenerate `wiki/evidence/actors/*.md`; write nothing on a failed reconciliation. Not: synthesis.
accepts:    Tests cover: a page whose envelope `count` exceeds its items (the old rule would pass, the new fails), total drift, a page-cap change, the id diff, the header fields; by hand, one real pull writes a snapshot whose header numbers add up and ≥ 300 actor pages.
verify-by:  `uv run pytest tests/test_store.py -q`; then `uv run foundry pull` (one-off) with the header pasted.
status:     todo

### S3.2 — Store-gaps table
needs:      S3.1
scope:      `fn store_gaps`: the ranked table from snapshot fields (notice, failed+timed-out share, rating vs category median, 30-day users vs previous snapshot, deprecated incumbents) written into `wiki/synthesis/store-gaps.md` under its prose; the prose grade column left for synthesis. Not: candidate scoring.
accepts:    On the planted snapshot fixture the table ranks the planted weak actor first; on the real snapshot it has ≥ 1 row.
verify-by:  `uv run pytest tests/test_store_gaps.py -q`; the real table's first three rows pasted.
status:     todo

### S3.3 — `wiki-pull.yml`
needs:      S3.2, S2.3
scope:      Weekly cron off the hour; `uv run foundry pull`; commits under `wiki/evidence/**` and `wiki/synthesis/store-gaps.md` only; no secrets. Not: notifications.
accepts:    A `workflow_dispatch` run commits a snapshot and the table.
verify-by:  `gh workflow run wiki-pull.yml && gh run watch` (one-off) → run URL and commit SHA pasted.
status:     todo

### S3.4 — Obsidian reads the wiki
needs:      S3.1
scope:      Open `wiki/` as a vault; Obsidian Git plugin with its base path at the repo root (or record that it cannot and the fallback is `git pull`); confirm actor pages and `[[links]]` render. Not: Obsidian Sync (never on `wiki/`).
accepts:    The vault opens on the latest snapshot's pages after a pull; the base-path premise is settled either way.
verify-by:  Manual (third-party): a screenshot or a one-line reading pasted, with the plugin setting named.
status:     todo

gate: OPEN — suite, diff review, drift check; milestone 1b done when S3.3 passes; user go.

## Stage 4 — Target research workflow and the interim sequencer (milestone 1c)
The first OS-shaped workflow runs end to end on its planted set, then once for real, and the
first candidate goes through the user's review. Proves ADR 011, 013 §2, 016 §1–4, 006.

### S4.1 — Interim sequencer `foundry run-workflow`
needs:      S1.2
scope:      `fn run_workflow`: load `workflow.yaml`, bind slots from `template.yaml` (refuse on a missing required slot), run each `kind: agent` stage in a fresh `claude -p` session with the body file, the bound slots, the input contract document and the tools from `.os/tools.yaml` (web reads as raw fetches, never a summariser), validate every output against its contract (stop, write nothing on failure), run `checks.yaml`, let the finisher open the PR with the five trailers and write the receipt to `.os/runs/`; `--fixture <dir>` substitutes planted inputs and a fake runner. Not: the OS shell's protocols.
accepts:    With a fake runner: a failed handoff stops the run with nothing written; a missing required slot refuses to start; the receipt names every stage and its contract result.
verify-by:  `uv run pytest tests/test_sequencer.py -q`
status:     todo

### S4.2 — Planted set and per-agent cases
needs:      S4.1
scope:      `library/wf.foundry.target-research/tests/planted/` as its README specifies (G1–G5 plus the injection page); per-agent case files; the plain-call baseline prompt. Not: real web reads.
accepts:    The fixture run yields exactly one note (G1), `your_changes_empty` true, the injection text appears only as a quotation; recall 5/5 against the planted list, unplanted findings reported separately.
verify-by:  `uv run foundry run-workflow wf.foundry.target-research --fixture library/wf.foundry.target-research/tests/planted/` → the receipt pasted; `uv run pytest tests/test_target_research_planted.py -q`
status:     todo

### S4.3 — First real run, on demand, locally
needs:      S4.2, S3.2, S2.3
scope:      Run with real slots (`max_notes: 2`); the PR opens with trailers; the daily check pings Telegram with the lines; the user merges, edits a note in Obsidian, sets `status`, commits; the next session writes the `decisions.md` ruling line, `reviewed`/`ruling` frontmatter and the rulings-log row. Not: a cloud routine (a later reading).
accepts:    PR URL; the Telegram ping text; `wiki/candidates/<slug>.md` on `main` with a status; one rulings-log row with four sub-scores and `match`.
verify-by:  The PR URL, the ping text and `tail -n 2 wiki/candidates/rulings-log.md` pasted.
status:     todo

### S4.4 — Baseline comparison
needs:      S4.3
scope:      The plain-call baseline (one prompt, one session, same slots) run once and compared on the planted set and the real run: notes that validate, exclusions honoured, saturation counted. Not: tuning.
accepts:    `library/wf.foundry.target-research/tests/baseline.md` records both results side by side with a verdict.
verify-by:  The file's verdict line pasted.
status:     todo

gate: OPEN — suite, diff review, drift check; milestone 1c done when S4.3 passes; user go.

## Stage 5 — Build workflow and actor #1 (milestone 2)
The builder as an OS-shaped workflow, the lint complete, the first real fixture, the review,
the private deploy, KYC, the publish runbook, the first daily test and Console Monday. Proves
ADR 014 and 015 §3–6 and the brief's publish gate.

### S5.1 — `library/wf.foundry.build/` and the template completed
needs:      S4.1, S1.6
scope:      `workflow.yaml`, `template.yaml`, `lens.md`, `checks.yaml`, bodies (scout — reads only the note's named sections, B20; implementer; docs — README to the contract; finisher), contracts for the handoffs, tests README; the actor template gains `listing.md` and the README template with the required sections. Not: actor #1's code.
accepts:    Bodies pass `tests/test_bodies.py`; the workflow's planted set (a small approved note + fixture) produces a PR whose diff touches only `actors/<name>/**` under the fake runner.
verify-by:  `uv run pytest tests/test_bodies.py tests/test_build_planted.py -q`
status:     todo

### S5.2 — `foundry lint` complete
needs:      S1.6
scope:      README structure (H2/H3 only, required sections in order), Output keys == `fields`, every input field described, prefill/default fixture-derived, dataset schema == `fields`, `listing.md` lengths and slug rule, cost example == FREE-tier `pricingInfos` (deploy only). Never word counts or keywords. Not: content judgement.
accepts:    One test per rule with a planted violation each; CI red on a planted README H1.
verify-by:  `uv run pytest tests/test_lint.py -q`; the red CI check URL pasted.
status:     todo

### S5.3 — Fixture for actor #1
needs:      S4.3 (an approved candidate with a ruling line)
scope:      The builder drafts `source/`, `expected.jsonl`, `fixture.md` (`status: draft`) from the note's **Suggested fixture page**; the user marks stable anchors and approves in their own commit. Not: the actor's code.
accepts:    `fixture.md` reads `status: approved` with `approved_on`; L1 green; anchors cover at least one `exact`, one `pattern`, one `present`.
verify-by:  `uv run pytest tests/test_fixture.py -q -k <actor>` and `git log -1 --format=%an -- actors/<actor>/fixture/fixture.md` showing the user.
status:     todo

### S5.4 — Build, review, private deploy
needs:      S5.1, S5.2, S5.3
scope:      `foundry run-workflow wf.foundry.build` opens the PR listing the B-IDs applied; `/code-review high` in a fresh session with the note, fixture and IDs as intent; `/red-team`; lint green; the user merges → deploy (private build), L1+L2 on the deployed build, read-backs green. Not: `isPublic`.
accepts:    PR URL; review record with findings dispositioned; deploy run green; `GET /v2/acts/{id}` shows the private actor with limited permissions.
verify-by:  The three URLs pasted; `uv run pytest -q` green at the merge commit.
status:     todo

### S5.5 — KYC, publish request, ruling, runbook, first daily test, one task
needs:      S5.4
scope:      KYC passed on the account (user) and recorded in `listing.md`; the publish request on Telegram (fixture passing, sample output vs the note, lint green, prefill duration and rows from the stub-style measurement on this actor, limited permissions, KYC date); the ruling line in `decisions.md`; the runbook's Console steps (price from measured CU vs the incumbent's FREE tier, SEO fields, categories, `isPublic`); `listing.md` committed with `#<PR>`; after Apify's first daily test passes (`notice: NONE` the next day), one task example published. Not: tiers, more tasks.
accepts:    `GET /v2/acts/{id}` public with `notice: NONE`, `actorPermissionLevel` limited, `pricingInfos` matching the README; the task page exists with view `overview`.
verify-by:  One-off, third-party: the API response and the task URL pasted; `uv run foundry lint actors/<actor>` green.
status:     todo

### S5.6 — First Console Monday row
needs:      S5.5
scope:      The Monday after publish: profit, score trend, open issues, new reviews → one row in `wiki/first-party/<actor>.md`; the Monday summary listed it as due. Not: automation of any of it.
accepts:    One dated row with the six columns; the summary message mentioned it.
verify-by:  `tail -n 1 wiki/first-party/<actor>.md` pasted with the summary text.
status:     todo

gate: OPEN — suite, diff review, drift check against the brief's boundary (actors sold through the Store, nothing bespoke); milestone 2 done when S5.5 passes; user go.

## Stage 6 — Actor #2 and the synthesis workflow (milestones 3 and 1d)
### S6.1 — `library/wf.foundry.wiki-synthesis/` and the first synthesis
needs:      S3.3 (two snapshots exist), S4.1
scope:      The workflow directory (reader, extractor, writer, finisher; contracts; planted snapshot with known weak actors and one known gap); first run through the sequencer: every `wiki/synthesis/` page filled with graded, linked claims; prose grades on store-gaps rows; at least one playbook proposal — all as PRs. Not: direct commits.
accepts:    Planted test surfaces the planted weak actors and gap; the PRs' pages carry a grade and a link on every claim (a test greps for ungraded lines).
verify-by:  `uv run pytest tests/test_synthesis_planted.py tests/test_synthesis_pages.py -q`; the PR URLs pasted.
status:     todo

### S6.2 — Actor #2 by the same route
needs:      S5.6, S4.3 (a second approved candidate)
scope:      S5.3–S5.6 repeated for the second candidate. Not: anything new in the apparatus; a needed change is a Revision.
accepts:    As S5.3–S5.6 for actor #2; both actors green in `STATUS.md`.
verify-by:  The same evidence set as Stage 5, pasted.
status:     todo

gate: OPEN — suite, diff review, drift check; milestones 3 and 1d; user go.

## Stage 7 — Thirty unattended days and the repair workflow (milestone 4)
### S7.1 — `library/wf.foundry.repair/` and `maintainer-repair.yml` — event-gated
needs:      the first real escalation (an L1 failure or an uncovered-page L2 failure on a live actor)
scope:      Built only once that escalation has happened and been repaired locally with `foundry repair`: the workflow directory (reproducer = the failing fixture result, implementer, fixer two rounds, finisher), in-job L1/L2 into the PR body, the Telegram repair line, the fixture draft when old L1 cannot pass; `maintainer-repair.yml` on `workflow_dispatch` with `CLAUDE_CODE_OAUTH_TOKEN` added to the secrets. If no escalation occurs by the end of S7.2 this step stays `todo` and the done-check records it as not yet in scope. Not: automatic dispatch (the trust ladder decides later).
accepts:    The first workflow-made repair PR carries L1/L2 results and the trailers; merging it (with the fixture approved on the branch) un-pauses in one deploy.
verify-by:  The PR URL and the deploy run URL pasted.
status:     todo

### S7.2 — Thirty consecutive days
needs:      S6.2
scope:      No hand-run checks; both actors' logs; at least one repair or named escalation in the window. Not: new actors.
accepts:    Thirty consecutive Actions-bot dates in each log; the window contains a repair (deploy un-pause line) or an escalation line with a named structural cause.
verify-by:  `grep -c '^2026-' maintainer/log/<actor>.md` for both, and the repair or escalation line pasted.
status:     todo

### S7.3 — Done-check against the brief
needs:      S7.2
scope:      Each statement under the brief's `## Done looks like` judged met / not met / not yet in scope with evidence: two actors public, pay-per-event, limited permissions; fixture per actor approved and passing on publish day; 30 unattended days with a repair or named escalation; every publish preceded by a ruling and a reviewed PR; at least one paid run by a stranger (Analytics, manual reading). Not: a mushy verdict — "sort of met" is a finding about the brief.
accepts:    The five-row table in this file with evidence per row.
verify-by:  The table pasted under this step; the paid-user count from Analytics with its read date.
status:     todo

gate: OPEN — final gate with the done-check; on all-met, the venture page's `state: shipped` routes through `/consolidate` (said here, not written).

## Routing sweep (every upstream commitment → where it is accepted)
- ADR 001 runner topology → S2.2, S2.5, S7.1 · ADR 002 fixture → S1.4, S1.6, S5.3 · ADR 003/012 pause → S1.3, S1.7, S1.9 · ADR 004 approval/deploy → S2.4 · ADR 005 log → S1.8, S2.2, S2.6 · ADR 006 layout → S1.1, S1.6, S1.7 · ADR 007/013 research runners and sequencing → S3.1–S3.3, S4.1–S4.4, S6.1, S7.1 · ADR 008 pricing events → S1.5, S1.7, S5.5 · ADR 009 preflight → S1.5, S1.7, S1.8 · ADR 010 re-approval gate → S2.4, S7.1 · ADR 011 branch rules → S2.3, S2.4, S4.1 (trailers) · ADR 014 congruency → S1.6, S2.5, S5.2 · ADR 015 operating → S1.8, S1.9, S5.5, S5.6, S7.1 · ADR 016 OS fit → S1.1, S1.2, S4.1, S5.1, S6.1, S7.1.
- Failure modes (architecture.md table) → the fake-based tests of S1.3, S1.5, S1.8, S2.4, S3.1, S4.1 and the stub checks of S1.7, S1.9.
- Verification posture → the suite accumulates from S1.1; one-offs are named where the posture says a test cannot hold the claim (S1.7, S2.1, S2.3, S3.4, S5.5, S7.3).
- Brief "Done looks like" → S7.3 row by row; brief failure posture → S1.5, S1.8, S1.9; brief boundary → Stage 5 and 6 gates' drift check.
- Plan risk rows → the assumed-premises list above; milestone slip dates → Stage 1–2 by 2026-11-15, Stage 5 publish and Stage 6 by 2026-12-31.
Unrouted after the sweep: none found. Not routed by design: tiered discounts, more task examples, Rising Star (deferred by ADR 015 §8).

## Revisions
- 2026-10-19 — r1: initial plan, drafted for the user's approval; Stage 1 is the walking skeleton on a private stub.
