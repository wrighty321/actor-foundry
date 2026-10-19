# Publish runbook

The user's steps, in order, from the publish ruling to the first Console Monday. This page is
playbook: it changes only through a pull request the user approves. Everything the deploy
workflow can read back, it does (ADR 014, 015); everything below is a Console action no
workflow performs.

## Before the ruling
1. The builder PR passed `/code-review high` in a non-authoring session and `/red-team`;
   `foundry lint` is green; the fixture reads `status: approved`.
2. The stub week recorded: the prefill run (the fixture's `live_pages[0]` with `maxItems =
   min_rows`) finished `Succeeded` with a non-empty dataset in under 2 minutes; the fixture run's
   compute units per 1k rows.
3. **KYC is passed** on the Apify account and the date is in `actors/<name>/listing.md`
   (`kyc_passed_on`). Without it the actor is excluded from default Store search and from
   `search-actors`, so nothing below is worth doing.
4. The publish request on Telegram shows all of the above plus a sample of real output against
   the approved candidate note. The ruling is a dated line in `context/decisions.md`.

## At publish (Console)
5. **Price.** Set the `result` event's price from the measured compute units against the
   incumbent's FREE-tier per-1k price (ADR 008); keep `apify-actor-start` at Apify's default;
   remove `apify-default-dataset-item`. No tiers yet (ADR 015 §8). Write the price into the
   README's cost example and `listing.md`; lint and deploy check they agree.
6. **Listing fields.** Title (40–50 characters, `{Site} {Service} Scraper`), description
   (≤ 300), SEO title (≤ 60), SEO description (145–155), categories — exactly as `listing.md`
   records them; the slug is now fixed (`published_on`).
7. **Permissions.** Confirm limited permissions; the actor is excluded from per-request agentic
   payments otherwise.
8. **Visibility.** Flip `isPublic`. Record `published_on` in `listing.md` and commit it (a
   `listing-change` PR with the ruling's `#<PR>` line).

## The first fourteen days
9. After the first daily test passes, publish **one** task example (Console or `apify task
   publish`): named as the user's goal made specific by place, industry or workflow; input as a
   small real job; dataset view `overview`. More tasks after 14 clean daily tests (ADR 015 §8).
10. Reply to any issue the same day; this is service to buyers, and the Store page shows the
    response time.
11. Watch `maintainer/STATUS.md` for Apify's `notice`; a break is repaired by `foundry repair`
    and a PR, with the fixture draft approved on the branch so one merge un-pauses.

## Console Monday (from the Monday after publish, about fifteen minutes)
12. Read, per actor: profit (Insights → Analytics), the quality score's **trend** (Insights →
    Actor quality — not its recommendations, which linger after fixes), open issues, new
    reviews. Reply to reviews where a reply helps the next buyer.
13. Append one row to `wiki/first-party/<actor>.md`:
    `| date | score | notice | open issues | new reviews | profit | note |`.
14. The Monday Telegram summary lists this step as due until the row exists.

## Never
- Never delete a public actor; deprecation is a human ruling in `decisions.md`, never the
  maintainer's write.
- Never use the manual *Under maintenance* toggle; it carries the same ranking penalty as the
  automatic label, and the actor's own status message already says what is wrong.
