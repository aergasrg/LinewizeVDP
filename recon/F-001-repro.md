# F-001 live repro plan — Content Blocked fail-open

**Rules of the road:** own managed device + own test account only. Pick a
**benign** blocked category to prove the point (a proxy/anonymizer, a blocked
game/streaming site, etc.) — never sexual content, never CSAM-adjacent, never
another user's account. If any other user's data appears: stop, report, extract
nothing. Screen-record everything; redact org/appliance IDs before submitting.

## Where the logs are

- **Content script logs** ("Hiding page…", "Verdict received:", "showing page
  again", "Requesting verdict…") → the **page's** DevTools console.
- **Background logs** (verdicts, block decisions) → `chrome://extensions` →
  Linewize Connect → **service worker → inspect**.
- If the managed policy disables DevTools (`DeveloperToolsDisabled`), skip to the
  purely visual tests — you can still see the page flash and screen-record it.

## Test A — is the block just a coat over a chair? (fastest)

1. Navigate to a benign blocked URL. The white overlay appears.
2. Open DevTools → Elements. Look for `<div id="linewize-protect">` at the end of
   `<body>`. Confirm the **real page is fully rendered underneath it**.
3. Console: `document.getElementById('linewize-protect').remove()`
4. If the page is now readable → enforcement was overlay-only for this state.
   (Also try Reader Mode / `Ctrl+U` view-source as no-DevTools variants.)

Proves primitive #2. Record it.

## Test B — the 5.5s fail-open (the core finding)

Goal: make the async verdict not arrive within `whitePageRemoveMs` (5500ms), and
watch the non-sexual page auto-reveal.

Cold-worker / uncached path (most realistic, no infra interference):
1. `chrome://extensions` → inspect the service worker.
2. Force it idle/stopped (MV3 evicts after ~30s idle; you can also hit **Stop**).
3. Immediately navigate to a **fresh, never-visited** benign blocked URL (add a
   junk `?x=<random>` query so it's uncached).
4. Watch: request goes through (Network tab), overlay shows, and if the verdict
   round-trip exceeds 5.5s the overlay is removed and the page shows. Record the
   console line "showing page again" firing **without** a BLOCK verdict.

Throttled-verdict variant:
- DevTools → Network → set throttling to a very slow profile (or "Offline")
  right as you navigate. Offline also exercises the `CHECK_INTERNET_CONNECTION →
  ALLOW` fail-open path noted in F-001. Observe whether the page reveals.

Repeat 5–10× to show it's reliable, not a fluke. Note which categories reveal
(hypothesis: everything the naive `porn/sex/adult/fuck` fallback doesn't catch).

## Test C — hider exhaustion (optional)

Cycle hide/unhide (navigate + back) rapidly and watch `blockCounter`; the hider
stops re-applying after `maxPageHideAttempts = 5`. Confirm whether that leaves a
window where the page stays visible.

## What to capture for the submission

- `.mp4` of B: uncached benign blocked URL loading through after ~5.5s.
- Console excerpt showing the hide→(no verdict)→unhide sequence.
- Note the exact category/URL and that no other user's data was involved.
