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
