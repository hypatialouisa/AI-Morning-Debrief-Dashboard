# Morning collection

Read `CLAUDE.md` in this repo first for the full spec (schema, sections, sources, constraints). This file is the step-by-step procedure for one collection run. Follow the Collection rules section of CLAUDE.md; this expands each rule into concrete actions.

Run this in a checkout of the project repo. If `data/` isn't present, something is wrong, stop and report it rather than inventing a folder layout.

## 1. Get the real current time

Run `date` (or equivalent) to get today's actual date and time in US/Eastern. Do not infer the date from conversation context or a previous run. All dates and windows in this run use Eastern time.

## 2. Gap check

Read `data/index.json`. Find the last archived weekday entry. Compare it against today's date. Every weekday strictly between the last archived date and today, with no entry, is missed and needs backfilling.

If `data/index.json` doesn't exist yet or is empty, treat every entry in `data/daily/` (if any) as the record instead, or start fresh if there's nothing at all.

## 3. Backfill missed weekdays

For each missed weekday, oldest first:

- Window is `00:00:00` to `23:59:59` Eastern on that date, unless the Monday rule (below) applies.
- Visit each approved source (`data/sources.json`, `status: "approved"`) for news published inside that window. This is real historical lookback, not a guess, use web search or the source's own archive/history to confirm actual publish dates.
- Write `data/daily/YYYY-MM-DD.json` for that date with `"backfilled": true`.
- Append the date to `data/index.json`.

## 4. Monday rule

If the file being written is for a Monday, its window starts at the Saturday before, `00:00:00` Eastern, not Monday `00:00:00`. This applies whether the Monday is today or a backfilled day.

## 5. Today's file

After backfill is caught up, create today's file:

- Window starts where the previous archived file's window ended, and ends now (the actual current time from step 1).
- `"backfilled": false`.
- Do not write today's file before backfill is complete, the window math depends on the previous file existing.

## 6. Visit every approved source

For each source in `data/sources.json` with `status: "approved"`:

- Try to fetch it. Record `{ "source": "<id>", "ok": true/false, "note": "" }` in `source_status`.
- If a source can't be reached (blocked, timeout, error), set `ok: false` and say why in `note`. Do not guess or fill that source's items from memory or from a different day.
- `openai-news` (openai.com/news) is known to block plain HTTP fetches with a 403. Try a browser-capable fetch tool if one is available; if not, fall back to web search for OpenAI's recent announcements and confirm the date and URL from the search result itself before writing an item. If neither works, mark it `ok: false` with that explanation, don't skip the note.
- For status pages (`status.openai.com`, `status.claude.com`), pull incidents and scheduled maintenance from the window and tag them `Outage`.

## 7. Write items

For each item found:

- `id`: first 12 characters of the sha1 hash of the canonical URL.
- `category` / `panel`: from which source and which list it came from in `data/sources.json` (`categories` and `panel` fields there). A source listed under multiple categories can produce items in any of them, use judgment on which category the specific item actually belongs to.
- `source_name`: the human name (e.g. "Anthropic", "TechCrunch"), not the source id.
- `summary`: your own words, one or two sentences. No pasted passages. A quote under 15 words is fine if the exact wording matters.
- `published_at`: must come from the source page itself. If you can't confirm it, leave the item out entirely, don't estimate.
- `type_tag`: exactly one of `Feature`, `Outage`, `Pricing`, `Policy`, `Legal`, `Research`, `Opinion`.

## 8. Deduplicate

Before adding an item, check whether its `id` already exists in any earlier file in `data/daily/`. If it does, skip it, it's already archived.

## 9. Never overwrite

If `data/daily/YYYY-MM-DD.json` already exists for a date you're about to write, leave it alone entirely. Move on.

## 10. A quiet day is fine

If a section or panel has nothing from the window, write an empty list for it. Don't pad with unrelated items to fill space.

## 11. Update the index

Append one entry per new file written (in date order) to `data/index.json`: `{ "date": "YYYY-MM-DD", "backfilled": true/false }`.

## 12. Log

Append one line to `logs/run-log.txt`:

```
2026-10-07T08:14:00-04:00 | wrote: 2026-10-06,2026-10-07 | items: 9,5 | failed: none
```

Run time, comma-separated dates written, item count per date, and any failed sources (or `none`).

## 13. Publish

`git add data/ logs/`, commit, and push, so the hosted dashboard picks up the new files. Use a plain commit message like `Add daily files for 2026-10-06, 2026-10-07`.

If this run has no network access to push (shouldn't happen, but just in case), still finish writing local files and say so clearly rather than silently skipping the publish step.

## 14. No fabrication, ever

Every item needs a real URL you actually found and a publish date you actually confirmed from the source. If you're not sure, leave it out. An empty section is always better than an invented one.

## 15. Report back

At the end, summarize: which dates were written, how many items per section, which sources failed and why, and anything that looked off (a date you couldn't confirm, a source that changed its page structure, anything that needs a human look).
