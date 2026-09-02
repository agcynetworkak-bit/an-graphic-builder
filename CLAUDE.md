# AN Graphic Builder — Agency Solutions LIVE calendar invite pages

This repo hosts **calendar-invite landing pages** for Agency Network's *Agency Solutions LIVE*
call series. Each page is a single self-contained HTML file (plus a matching `.ics`) that gets
posted to Facebook. When someone taps the link, they get a branded page with an **"Add to
Calendar"** button (Google Calendar, Outlook, Apple, `.ics` download, email-to-self, copy) and a
**"Join Microsoft Teams"** button.

The site is served by **Netlify**, which auto-deploys from the `main` branch of this repo, at:
`https://an-nasfa-group-health-plan-update.netlify.app/<file>.html`

> **GitHub Pages is no longer used.** GitHub is source control only. Do not quote a
> `agcynetworkak-bit.github.io` URL to the user and do not use the "pages build and deployment"
> workflow as proof a page is live.

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

Develop on the feature branch, then merge to `main` — Netlify builds on every push to `main`:
```
git add <files> && git commit -m "..."
git push -u origin <feature-branch>
git checkout main && git pull origin main
git merge --ff-only <feature-branch>
git push origin main
git checkout <feature-branch>
```

## ⚠️ MANDATORY: verify the link is LIVE before telling the user it's done

Netlify builds after every push to `main` and takes roughly a minute. During that window the new
URL 404s even though the file is correct. Do **not** report a page as ready until you have
actually confirmed it — and if you cannot confirm it, say so plainly instead of implying you did.

1. **Push to `main`**, then give Netlify ~60 seconds.
2. **Verify the URL loads (HTTP 200, not 404):**
   ```
   curl -sS -o /dev/null -w "%{http_code}" https://an-nasfa-group-health-plan-update.netlify.app/<file>.html
   ```
   Expect `200`.
3. **If a `NETLIFY_AUTH_TOKEN` is available**, confirm the deploy state instead:
   ```
   curl -sS -H "Authorization: Bearer $NETLIFY_AUTH_TOKEN" \
     "https://api.netlify.com/api/v1/sites/<site-id>/deploys?per_page=1"
   ```
   Look for `"state":"ready"` on a deploy whose `commit_ref` matches your push.

### When you cannot verify (common in the sandbox)

The agent proxy currently **blocks `netlify.app` and `agcynetwork.com`** (`CONNECT tunnel failed,
403`), and there is no Netlify CLI or token in the environment. When that is the case there is no
way to confirm the page is live from here.

Say exactly that. Tell the user the file is pushed and Netlify should publish within a minute, and
ask them to confirm. **Never state or imply a link is verified live when the check was blocked**,
and never substitute a GitHub signal as proof — GitHub knows nothing about the Netlify deploy.

A 404 right after a push is expected — reload after a minute, or add `?v=1` to bust the phone /
Facebook in-app-browser cache.
