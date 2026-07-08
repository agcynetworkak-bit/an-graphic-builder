# AN Graphic Builder — Agency Solutions LIVE calendar invite pages

This repo hosts **calendar-invite landing pages** for Agency Network's *Agency Solutions LIVE*
call series. Each page is a single self-contained HTML file (plus a matching `.ics`) that gets
posted to Facebook. When someone taps the link, they get a branded page with an **"Add to
Calendar"** button (Google Calendar, Outlook, Apple, `.ics` download, email-to-self, copy) and a
**"Join Microsoft Teams"** button.

The site is served by **GitHub Pages from the `main` branch**, at:
`https://agcynetworkak-bit.github.io/an-graphic-builder/<file>.html`

## How to build a new call's page

1. **Copy the most recent page as the template.** Use the latest `asl-*.html` +
   `asl-*.ics` pair (e.g. `asl-sean-kingsbury-marketing-executive.*`) — the layout, styling,
   and JS are identical every time; only the content and meeting data change.
2. **File naming:** `asl-<speaker-or-topic-slug>.html` and `.ics` (kebab-case). The `.ics`
   filename must match the `webcal://`, download `href`, and `.ics` UID references inside the HTML.
3. **Fill in the event data** in both files: title, speaker + role, date, description,
   "What we'll cover"/panel bullets, Teams join URL, Meeting ID, passcode.
4. **The `<script>` `cfg` object is the single source of truth** for every calendar button URL.
   Update `title`, `startUTC`, `endUTC`, `startISO`, `endISO`, `teamsURL`, `meetingID`,
   `passcode`, `location`, and the `cfg.body` lines.

### Data / conventions
- **Times are Eastern.** Standard call window is **1:00–2:30 PM Eastern** unless told otherwise
  (the extra 30 min is intentional buffer). Graphics often say "EST" even in summer — that's fine,
  but the event must be anchored to the correct **UTC** offset for the actual date:
  - EDT (mid-Mar → early Nov): 1:00 PM ET = **17:00 UTC**, 2:30 PM ET = **18:30 UTC**.
  - EST (winter): 1:00 PM ET = **18:00 UTC**, 2:30 PM ET = **19:30 UTC**.
  Always double-check the day-of-week against the date; note the correct one on the page.
- `.ics` files must keep **CRLF** line endings (enforced by `.gitattributes` — do not "fix" them).
- Some invites include a Meeting ID + passcode; some only have a `meetup-join` URL. Show the
  Meeting ID/passcode block only when they exist.

## Publish flow (branch → main → live)

Develop on the feature branch, then merge to `main` so Pages publishes it:
```
git add <files> && git commit -m "..."
git push -u origin <feature-branch>
git checkout main && git pull origin main
git merge --ff-only <feature-branch>
git push origin main
git checkout <feature-branch>
```

## ⚠️ MANDATORY: verify the link is LIVE before telling the user it's done

GitHub Pages runs a **"pages build and deployment"** workflow after every push to `main`, and it
takes ~60 seconds. During that window the new URL returns **404** even though the file is correct.
Do **not** report the page as ready until the deploy has finished. Every time:

1. After pushing to `main`, **wait for the Pages deploy to complete successfully.** Check the
   latest `pages build and deployment` run via the GitHub Actions API and confirm
   `status: completed` / `conclusion: success` (it started at/after your push time):
   ```
   mcp__github__actions_list  method=list_workflow_runs branch=main event=dynamic
   ```
   (Outputs are large — parse the saved JSON for the top run's status/conclusion/created_at.)
2. **Then verify the URL actually loads (HTTP 200, not 404).** Try:
   ```
   curl -sS -o /dev/null -w "%{http_code}" https://agcynetworkak-bit.github.io/an-graphic-builder/<file>.html
   ```
   Expect `200`. If the environment blocks the fetch, rely on the successful Pages deploy as
   confirmation and say so.
3. **Only after the deploy is green (and ideally a 200) do you tell the user the link is ready.**
   Never hand over a link while the build is still `in_progress`.

If you must report before the build finishes, say explicitly that Pages is still deploying and to
wait ~1–2 minutes (and that a 404 during that window is expected — reload, or add `?v=1` to bust
the phone/Facebook in-app-browser cache).
