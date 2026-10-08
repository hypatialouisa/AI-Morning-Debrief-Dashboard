# AI Morning Debrief Dashboard

## Goal

Build and run a daily AI news dashboard. The user opens it by 9:00 AM Eastern each weekday. It shows what changed overnight on three platforms, plus outside coverage and practice news. Every day is archived.

Personal project, personal laptop, personal Claude account. No firm or IT approval needed.

The dashboard itself is a browsing and capture tool: browse by day, section, and tag, check items and jot a one-line note as you read, then download that day's or a date range's data as JSON with your checks and notes merged in. It is hosted on GitHub Pages as static files. No server-side code, no API keys, no accounts to configure, and nothing is saved automatically: a refresh clears whatever you haven't exported yet, by design.

The weekly newsletter draft is written outside this project, using Claude Cowork on a separate computer. Checking items and jotting notes happens in the dashboard itself; only the final drafting step moves to Cowork. See "Weekly notes and summary" below. This project's job stops at collecting items, publishing the dashboard, and letting you export what you've flagged.

Repo: https://github.com/hypatialouisa/AI-Morning-Debrief-Dashboard (public). Live at https://hypatialouisa.github.io/AI-Morning-Debrief-Dashboard/dashboard.html, hosted on GitHub Pages from the `main` branch root.

## Folder layout

```
/ai-debrief
  CLAUDE.md          this file
  collect.md         the morning collection prompt (create in build step 3)
  dashboard.html     single-file page, published to GitHub Pages
  /data
    sources.json     approved source list (create in build step 1, local only)
    index.json        list of dates with a daily file, updated each run (published)
    /daily
      2026-10-06.json    one file per weekday, never overwritten (published)
  /logs
    run-log.txt      one line per run: time, days filled, errors (local only)
```

`data/daily/*.json`, `data/index.json`, and `data/sources.json` are committed and pushed, so the published page can fetch them (the footer reads `sources.json` to list where everything comes from). `logs/run-log.txt` stays local, the page never needs it.

## Sections

Five sections. Each one has two panels.

| Key | Section | Panel A "From the source" | Panel B "Outside coverage" |
|---|---|---|---|
| legora | Legora | Legora's own announcements | Reputable outside coverage of Legora |
| chatgpt | ChatGPT | OpenAI's own announcements, release notes, status | Reputable outside coverage of ChatGPT |
| claude | Claude | Anthropic's own announcements, release notes, status | Reputable outside coverage of Claude |
| legal_ai_practice | Legal AI practice | (none, see note) | Legal tech trade press, law firm adoption news |
| best_practices | Best practices | Official prompting guides | Expert writing on workflows and prompting |

In the JSON, the panel value is `source` or `outside`.

Legal AI practice has no dedicated Panel A source. Bar and ABA guidance is rare and shows up in trade press when it matters, so it's not tracked separately; Panel A stays empty for this section unless that changes later. Panel A for Best practices holds the Anthropic and OpenAI prompting guides.

Content that is platform-specific (a Claude or ChatGPT prompting tip published by Anthropic or OpenAI themselves) goes in that platform's own Panel A, not in Best practices. Best practices is for general, platform-agnostic workflow and prompting guidance.

Legora's Panel B draws from both the general outside-coverage list and the Legal AI practice trade press list.

## Approved sources

The user approved this list. Do not add sources without asking. All sources must be free to read. Remove any source that blocks reading behind a paywall or login.

### From the source

- Legora: company blog and news page. (LinkedIn removed, not a good source. Check during build step 1 for a separate changelog or release-notes page; if none exists, the blog alone is enough.)
- ChatGPT: OpenAI news, ChatGPT release notes, status.openai.com
- Claude: Anthropic news, Claude release notes, status.claude.com

### Outside coverage, general

- The Verge
- Ars Technica
- MIT Technology Review (mixed model, some feature pieces are subscriber-only even though the tested article was free; skip individual paywalled pieces rather than dropping the source)
- TechCrunch
- VentureBeat

### Legal AI practice

- Artificial Lawyer
- Legal IT Insider

### Best practices

- Ethan Mollick, One Useful Thing
- Simon Willison's blog
- Anthropic and OpenAI official prompting guides
- Stanford HAI

### Removed because paywalled

Bloomberg, Financial Times, Wall Street Journal, Bloomberg Law, Reuters, Wired, Law.com.

### Paywall verification (build step 1)

Done. Results are in `data/sources.json` with a `verified_on` date on each source. Final list approved.

## Item tags

Each item gets exactly one `type_tag`:

`Feature`, `Outage`, `Pricing`, `Policy`, `Legal`, `Research`, `Opinion`

- Use Outage for maintenance notices and incidents from status pages.
- Use Legal for litigation, court rulings, and bar rules.
- Use Policy for regulation and company policy changes.

## JSON schema

### Daily file: `data/daily/YYYY-MM-DD.json`

```json
{
  "schema_version": 1,
  "date": "2026-10-06",
  "generated_at": "2026-10-06T08:14:00-04:00",
  "window": {
    "start": "2026-10-03T00:00:00-04:00",
    "end": "2026-10-06T08:00:00-04:00"
  },
  "backfilled": false,
  "source_status": [
    { "source": "status.claude.com", "ok": true, "note": "" }
  ],
  "items": [
    {
      "id": "sha1 of the canonical URL, first 12 characters",
      "category": "claude",
      "panel": "source",
      "source_name": "Anthropic",
      "title": "Item title",
      "url": "https://...",
      "published_at": "2026-10-05T14:30:00-04:00",
      "type_tag": "Feature",
      "summary": "One to two plain sentences in the collector's own words."
    }
  ]
}
```

### Rules for items

- `id` is stable. The same URL always gives the same id.
- `summary` is written in the collector's own words. No pasted passages. Quote at most a short phrase, under 15 words, and only when the exact wording matters.
- `published_at` must come from the source page. If the date cannot be confirmed, leave the item out.
- Dates use Eastern time.

### Index file: `data/index.json`

```json
{
  "schema_version": 1,
  "dates": [
    { "date": "2026-10-06", "backfilled": false }
  ]
}
```

One entry per archived weekday, oldest first. The dashboard fetches this to know which days it can load and to draw the weekday strip and gap markers, since a static site can't list a folder's contents on its own. The collector appends to it in the same run it writes a new daily file.

## Export formats

- **Day export:** a copy of that day's file, with each item's `checked` and `note` merged in from your current session.
- **Range export:** an object with `from`, `to`, and an array of day exports, for grabbing a full week at once.

No import. Nothing persists between sessions, so there's nothing to merge on load, only on export.

## Collection rules (for collect.md)

Run each weekday starting at 8:00 AM Eastern. The page must be ready by 9:00 AM.

1. **Gap check.** Read `data/index.json`. Find the last archived weekday. Compare against today. Every weekday between them with no entry counts as missed.
2. **Backfill.** For each missed weekday, in date order, create that day's file. Set `backfilled: true`. The window is 00:00 to 23:59 Eastern on that date.
3. **Monday rule.** The Monday file also covers Saturday and Sunday. Its window starts at Saturday 00:00 Eastern. If Monday itself was missed and is being backfilled, apply the same rule.
4. **Today.** Create today's file. The window starts at the previous file's window end.
5. **Deduplicate.** Skip any item whose id already exists in any earlier daily file.
6. **Per-source checks.** Visit each approved source. Record success or failure in `source_status`. If a source fails, say so in the file. Do not guess or fill from memory.
7. **Never overwrite.** If a day's file exists, leave it alone. Write new files only.
8. **Status pages.** Capture incidents and scheduled maintenance from the window. Tag them Outage.
9. **Update the index.** Append each new day to `data/index.json`.
10. **Log.** Append one line to `logs/run-log.txt`: run time, days written, item counts, failed sources.
11. **Publish.** Commit and push the new daily files and the updated index so the hosted page picks them up.
12. **No fabrication.** Every item needs a real URL and a confirmed publish date.

A day with no news in a section is valid. Write an empty list for it.

## Weekly notes and summary (outside this project)

Checking items and jotting notes happens in the dashboard itself, in page memory only. Nothing is saved until you export. Writing the actual weekly draft happens separately with Claude Cowork, running on a different computer, pointed at its own local folder, not this project's `/data`.

Workflow:

1. As you read each day, check items worth keeping and jot a one-line note directly on the dashboard.
2. Before you move on, export that day or the week's range as JSON. This is the save step. A refresh before exporting loses that session's checks and notes.
3. Drop the exported file into the Cowork folder.
4. Friday, ask Cowork to read the week's file and draft the summary from your checked items and notes. Add anything extra in chat if you want, the notes are already there.

This project's data never needs to sync back with that folder.

## Scheduling

Confirmed: a cloud routine, created at claude.ai/code/routines, runs `collect.md` on a weekday schedule in Anthropic's cloud. Answers to the original questions:

- A cloud routine doesn't need this Mac open or awake. It runs on its own clone of the GitHub repo, not the local `/data` folder. (The desktop app can also run local scheduled tasks, but those only fire while the app is open, so the cloud routine is the better fit here.)
- It can fetch the web and write, commit, and push files inside its own sandboxed clone, no local prompts.
- Minimum schedule granularity is 1 hour, fine for once per weekday.

Because of this, the published repo is the canonical copy going forward. This Mac's local `/data` folder was useful for the build and samples, but won't auto-update once the routine is running, only `git pull` refreshes it.

Set up in build step 8: a routine pointed at this repo, running `collect.md` weekdays at 8:00 AM Eastern (converted to UTC when it's created).

To run it by hand: open it at claude.ai/code/routines and use "Run now," or run `collect.md` in a local Claude Code session in this folder. The gap check makes either option safe. A late or missed run never loses a day.

## Page spec: `dashboard.html`

One self-contained file. Plain HTML, CSS, and JavaScript. No external scripts, fonts, or images. Hosted on GitHub Pages; works in any modern browser, including on a phone.

### Data access

- On load, fetch `data/index.json`, then fetch each daily file it needs (the current day, the weekday strip, whatever range is in view).
- A fetch that 404s means that day has no file yet. Show it as empty in the weekday strip, not as an error.
- No folder picker, no stored permissions, no offline fallback. It's a static page reading static files from the same site.
- Checks and notes live in page memory only, never saved to the browser or anywhere else. A refresh or closed tab loses anything not yet exported. This is intentional, export is the save step, not a background autosave.

### Layout

- Header with the date picker, previous and next day buttons, and a "Today" button.
- A strip of the last 10 weekdays. Days with a file show filled. Days with no file show empty. This makes gaps visible.
- Five section tabs. Each tab shows two panels, visually separate but side by side: "From the source" and "Outside coverage."
- Each item shows: type tag, title linked to the source, source name, publish time, summary, a checkbox, and a note field.
- Filter by type tag across the current day, plus a "Select all" control that marks every tag active in one click (and clears them all if every tag is already active).
- Show `source_status` failures in a small notice at the top of the day.
- Newest items first within each panel.
- No second link on the source name, the title link to the specific article is enough.

### Footer

A small footer, same on every day, listing where everything comes from: fetch `data/sources.json`, filter to `status: "approved"`, group by `name`, one link per name to that source's first listed URL.

Not part of any export. Day and range exports are daily-file data only; the footer is static page chrome.

### Export buttons

- Export this day as JSON
- Export date range as JSON

### Writing rule

For any text the page generates: short sentences, no em dashes, no filler transitions.

## Visual style

A personal color and type palette, not tied to any company's brand guidelines.

### Color tokens

Started from a reference palette the user provided (an "Onyx Design" swatch set: Lavender Mist, Midnight Indigo, Crimson Bloom, Berry Noir, Deep Merlot), then revised per feedback: Policy and the shared hover-highlight color moved from Midnight Indigo to a vibrant blue, Legal moved from Berry Noir (too close to the background family) to a beige, Opinion moved from a yellow-green emerald to a blue-leaning teal, and Outage/Pricing both got more saturated. Crimson Bloom and Lavender Mist stayed from the original swatch set. The dark background is close to black with just a trace of wine in it.

Every pairing below is checked against WCAG 2.1 AA (4.5:1 for normal text, 3:1 for large text and UI fills). Where a single hue couldn't pass on both the dark and light canvas, there are two values, one per mode, both still clearly the same hue family.

```css
:root {
  --color-white: #FFFFFF;
  --color-black: #000000;
  --color-bg-dark: #170310;
  --color-bg-light: #F4F1F8;
  --color-lavender: #A59FF7;
  --color-crimson: #A40045;
  --color-crimson-bright: #F50067;
  --color-blue: #1C64F2;
  --color-blue-bright: #3777F4;
  --color-blue-deep: #145EF2;
  --color-beige: #C2A572;
  --color-orange: #E35D0F;
  --color-gold: #EDA711;
  --color-teal: #128F82;
  --color-dark-gray: #6E5F68;
  --color-mid-gray: #9A8792;
  --color-light-gray: #DCD5DE;
}
```

**Hierarchy:** a near-black canvas by default, white text, Crimson Bloom for filled accents (buttons, tabs, active states); a light-mode variant swaps to a lavender-tinted off-white canvas, black text, same Crimson Bloom fill. The other colors appear only in small elements such as tag chips and status marks, never as large backgrounds or primary text. White space is generous. No stroke/border on buttons, tabs, or tag chips, solid fills instead, flat outlines read as an afterthought against this palette.

### Dark mode

Dark is the default on load. A toggle in the header switches to light. Like checks and notes, the choice isn't saved, a refresh goes back to dark.

```css
:root {
  --bg: var(--color-bg-dark);
  --bg-raised: #2B0817;
  --text: var(--color-white);
  --text-secondary: var(--color-mid-gray);
  --accent: var(--color-crimson);
  --accent-hover: var(--color-blue);
  --accent-text: var(--color-white);
  --accent-hover-text: var(--color-white);
  --link: var(--color-crimson-bright);
  --link-hover: var(--color-blue-bright);
}
:root[data-theme="light"] {
  --bg: var(--color-bg-light);
  --bg-raised: #E7E2EE;
  --text: var(--color-black);
  --text-secondary: var(--color-dark-gray);
  --accent: var(--color-crimson);
  --accent-hover: var(--color-blue);
  --accent-text: var(--color-white);
  --accent-hover-text: var(--color-white);
  --link: var(--color-crimson);
  --link-hover: var(--color-blue-deep);
}
body { background: var(--bg); color: var(--text); }
```

Two different jobs were getting tangled under one "accent" idea, which is what caused the contrast failures: a color used as a **fill** (a button or chip's background, with white or black text on top of it) has a totally different contrast requirement than a color used **as text** directly on the page background. So they're split:

- `--accent` / `--accent-hover`: fill colors only (Crimson Bloom, the vibrant blue), same in both modes, always paired with `--accent-text` / `--accent-hover-text` so the label on top is always legible. White text passes on both: 7.9:1 on Crimson Bloom, 5.1:1 on the blue.
- `--link` / `--link-hover`: for a plain colored link or hover state sitting directly on the page canvas. Crimson Bloom itself fails badly as text on the dark canvas (2.5:1), so dark mode uses a brightened version instead. The blue fails as text on both canvases at full strength, so each mode gets its own adjusted version (brightened for dark, deepened for light), both still clearly the same blue.
- `--text-secondary`: meta text, captions, muted labels. A single gray can't clear 4.5:1 against both a near-black and a near-white canvas, so this reuses Mid Gray in dark mode and Dark Gray in light mode, both already in the palette.

**Typography:** Montserrat, self-hosted. Woff2 files live in `/fonts` in the repo, added in build step 4, not loaded from Google's CDN, so the page keeps working if that's blocked. Falls back to system sans-serif if the font file fails to load.

```css
@font-face {
  font-family: 'Montserrat';
  src: url('fonts/Montserrat-Regular.woff2') format('woff2');
  font-weight: 400;
}
@font-face {
  font-family: 'Montserrat';
  src: url('fonts/Montserrat-Bold.woff2') format('woff2');
  font-weight: 700;
}
body, p {
  font-family: 'Montserrat', Arial, Helvetica, sans-serif;
  font-weight: 400;
}
h1, h2, h3, h4, h5, h6 {
  font-family: 'Montserrat', Arial, Helvetica, sans-serif;
  font-weight: 700;
}
a { color: var(--link); }
a:hover { color: var(--link-hover); }
a:focus { color: var(--link); text-decoration: underline; }
```

### UI components

No border/stroke on buttons, tabs, or tag chips anywhere in this list, solid fills and color/weight changes carry the state instead.

- **Buttons:** `--bg-raised` fill, `--text` label. Hover: `--accent-hover` fill, `--accent-hover-text` label. Focus-visible: a 2px `--accent` outline (keyboard focus only, not a permanent stroke). Active/primary: `--accent` fill, `--accent-text` label.
- **Text links:** `--link`. Hover: `--link-hover`. Visited or focus: `--link` with underline.
- **Cards (each news item):** `--bg-raised` fill, no border. Hover: `--accent` fill, `--accent-text` label.
- **Tabs:** `--text-secondary` label on transparent, no fill, when inactive. Hover: `--bg-raised` fill, `--text` label. Active: `--accent` fill, `--accent-text` label.
- **Tag chips (both the filter row and the ones on each item):** always their assigned tag color as a solid fill (below) with whichever of black or white actually clears 4.5:1 on that specific color, never outlined. The active state in the filter row is bold plus underline, not a color change, dimming any of these chips for an "inactive" look pushed more than one below AA.
- **Accordions:** `--text` for text and rules. Hover: `--link-hover` text. Open: `--link` text.
- **Icons:** `--text`. Hover: `--link-hover`. Never same color as `--bg`.
- **Subtle backgrounds and dividers:** Light Gray (`#DCD5DE`), decorative only, not relied on to convey anything by itself.
- **Secondary text:** `--text-secondary` (theme-aware, see Dark mode above).
- **Disabled states:** Dark Gray (`#6E5F68`). WCAG exempts disabled controls from the contrast requirement, so this doesn't need to clear 4.5:1.

### Tag chip colors (small elements only)

Text color is whichever of black or white actually clears 4.5:1 against that fill, checked, not assumed.

| Tag | Color | Text |
|---|---|---|
| Feature | Crimson Bloom (`#A40045`) | White (7.9:1) |
| Outage | Orange (`#E35D0F`) | Black (5.8:1) |
| Pricing | Gold (`#EDA711`) | Black (10.1:1) |
| Policy | Blue (`#1C64F2`) | White (5.1:1) |
| Legal | Beige (`#C2A572`) | Black (8.9:1) |
| Research | Lavender Mist (`#A59FF7`) | Black (8.9:1) |
| Opinion | Teal (`#128F82`) | Black (5.3:1) |

### Charts

If any are added later: cycle colors in this order: Crimson Bloom, Blue, Gold, Orange, Teal, Lavender Mist, Beige. Use Crimson Bloom plus Gold for two series.

## Build order

1. **Sources.** Create `data/sources.json`. Run the paywall checks. Show results to the user and get approval of the final list.
2. **Schema and sample.** Create the folder layout. Write two sample daily files with realistic fake items, clearly marked `"sample": true`, plus a matching `index.json`.
3. **Collector.** Write `collect.md`. Check scheduling options in the current documentation. Report what is possible and let the user pick.
4. **Dashboard.** Build `dashboard.html` to the page spec and visual style. Test with the sample files.
5. **Publish and test.** Push to a GitHub repo, enable Pages, confirm the hosted page loads the sample data correctly, including on a phone.
6. **First real run.** Run the collector by hand. Review every item with the user. Check for bad dates, missing URLs, and sources that failed.
7. **Gap test.** Delete a real archived day's file and its index entry. Run the collector. Confirm it detects and fills the gap. (Originally planned against a sample file, but samples were removed early, see step 10.)
8. **Schedule.** Set up the chosen schedule for 8:00 AM Eastern, weekdays.
9. **Weekly test.** Check a few items and add notes on the dashboard, export a week's range, feed it to Cowork, and ask for the summary. Confirm it reads cleanly for email.
10. **Remove samples.** Done early, out of order: the two sample files (Oct 5, Oct 6) were confusing on the live site (their `example.com` links go to a registrar notice page) and real data already existed for Oct 7, so they were deleted right after step 6 rather than waiting for the end of the build.

## Constraints and cautions

- Personal laptop and account. Still ask before installing software or changing system settings.
- Keep news items and personal notes only. Nothing else.
- Ask the user before adding any dependency, source, or scheduled job.
- Show the plan and wait for the user to say "go ahead" before running any build step or collection run.

## Writing style for anything Claude writes for the user

- Lead with the point. No preamble. No restating the request.
- Short sentences.
- No "it is not X, it is Y" constructions. No stock openers. No closing summaries that repeat the body.
- No filler transitions such as "moreover" or "furthermore."
- Avoid these words: delve, leverage, robust, landscape, tapestry, foster.
- No em dashes by default.
- Use concrete details, real numbers, and named examples.
- No emojis unless asked.
- No praise of the user's questions. No stacked hedging. Use plain verbs.
- Ask clarifying questions before giving detailed answers.

## Open items

- None currently. Next up is build step 6, the first real collector run.
