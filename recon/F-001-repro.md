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

## No-DevTools environment (the realistic case)

On a locked-down managed Chromebook DevTools, `javascript:` URLs and bookmarklets
are all typically disabled (`DeveloperToolsDisabled`, `URLBlocklist`). **This
makes any bypass that still works here MORE severe** — it's reachable by a normal
student, not a researcher with a console. Prefer these tool-free methods; the
DevTools steps below are only for a machine where they happen to be allowed.

### Test P — Print-media overlay removal (no tools, from their own CSS)

The hider injects `@media print { #linewize-protect { display:none !important } }`
(`filter.bundle.js`, `hidePage`). The page is fully loaded underneath, so:

1. Open a benign blocked URL; the white overlay appears.
2. `Ctrl+P` (or menu → Print). In **Print Preview** the overlay is gone and the
   blocked page content renders.
3. "Save as PDF" to capture it. (If `PrintingEnabled` is off by policy, note that
   in the report — preview alone still demonstrates the reveal.)

This needs zero DevTools/`javascript:`/bookmarklets. Screen-record it.

### Test W — Wi-Fi/verdict stall (no tools)

Uses the OS network toggle (not a browser control, so browser policy can't block
it) to starve the async verdict → 5.5s fail-open / `CHECK_INTERNET_CONNECTION →
ALLOW` path.

1. Visit a benign blocked URL once so it's in disk cache; confirm it's blocked.
2. Turn **Wi-Fi off** from the ChromeOS system tray, then reload (or navigate via
   address bar with a cached page).
3. Watch whether the page reveals (verdict can't be fetched; fallback timer /
   offline-ALLOW). Toggling Wi-Fi off *during* load is the timing variant.

### Test U — Uncached URL + idle worker (no tools)

1. Leave the browser idle ~1 min so the MV3 service worker evicts.
2. In the address bar, open a **fresh** benign blocked URL with a random query
   (`...?x=93217`) so no cached verdict exists.
3. Watch for the page revealing after ~5.5s without a BLOCK. Repeat 5–10×.

Capture all three on video; note category/URL; no other user's data involved.

## Test A — is the block just a coat over a chair? (fastest, DevTools only)

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
