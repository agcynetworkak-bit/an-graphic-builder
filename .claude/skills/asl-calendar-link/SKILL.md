---
name: asl-calendar-link
description: Build an Agency Solutions LIVE calendar-invite landing page (+ .ics) for a new call, publish it to GitHub Pages via main, and VERIFY the live link works before reporting done. Use when the user pastes call details (speaker, topic, date, time, Teams link) and wants a Facebook-shareable "add to calendar" link.
---

# Agency Solutions LIVE — calendar invite link

Build a new calendar-invite landing page for an Agency Solutions LIVE call, following the exact
pattern of the existing `asl-*.html` / `asl-*.ics` pages in this repo.

## Steps

1. **Gather the call details** (ask only for what's missing):
   speaker + role, talk title, date, start–end time + time zone, Teams join URL, and (if present)
   Meeting ID + passcode. A graphic/flyer often has the date/time — read it.
2. **Copy the newest `asl-*.html` + `asl-*.ics`** pair as the template. Only the content and the
   `<script>` `cfg` object change between calls. Name new files `asl-<slug>.html` / `.ics` and make
   the `.ics` filename match every reference inside the HTML (`webcal://`, download link, UID).
3. **Fill in content + `cfg`** (single source of truth for all calendar button URLs): `title`,
   `startUTC`/`endUTC`, `startISO`/`endISO`, `teamsURL`, `meetingID`, `passcode`, `location`,
   `cfg.body`. Also update the `.ics` `DTSTART`/`DTEND`/`SUMMARY`/`DESCRIPTION`/`LOCATION`/UID.
   - Standard window is **1:00–2:30 PM Eastern**. Anchor to UTC by season: **EDT → 17:00/18:30 UTC**,
     **EST → 18:00/19:30 UTC**. Verify the weekday matches the date and note the correct zone.
   - `.ics` must stay CRLF (see `.gitattributes`).
4. **Publish:** commit on the feature branch, push, then merge to `main` and push so Pages builds it.

## ⚠️ Before you say it's done — VERIFY the link is live

The Pages **"pages build and deployment"** workflow runs ~60s after each push to `main`; during that
window the URL 404s even though the file is correct. Do not report the link as ready until:

1. The latest `pages build and deployment` run for `main` shows `status: completed` /
   `conclusion: success` — check via `mcp__github__actions_list method=list_workflow_runs
   branch=main event=dynamic` (parse the saved JSON; outputs are large).
2. Ideally also confirm the URL returns HTTP 200:
   `curl -sS -o /dev/null -w "%{http_code}" https://agcynetworkak-bit.github.io/an-graphic-builder/<file>.html`
   (If the environment blocks the fetch, treat a green Pages deploy as confirmation and say so.)

Only then give the user the link. If you report earlier, say Pages is still deploying, to wait
~1–2 min, and that a 404 in that window is expected (reload or append `?v=1` to bust caches).
