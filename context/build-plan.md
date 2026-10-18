# Build plan — actor-foundry

Living. Stages are appended; a stage's steps run one at a time, each verified exactly as its
`verify-by` says, with the evidence recorded under it. A failed verify stops the run. A step
that breaks something it cannot fix within its own stated scope is reverted and reported,
never widened. `/blueprint` owns the build stages for milestones 1–5 and should append them
below; `/upgrade` appends upgrade stages. The building happens in a separate session from the
planning (ruled by the user, 2026-10-18); a step marked **elsewhere** is executed there.

## Protocol
1. Read the step's `accepts` and `verify-by` before touching anything.
2. Make the change, run `verify-by` as written, paste what it returned under `evidence`.
3. Mark the step `done`, `blocked` (with what blocks it) or `reverted` (with why).
4. Close the stage with the integration check it names.

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
