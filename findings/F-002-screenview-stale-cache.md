# [F-002] Screen View can serve a stale (≤30s) frame under capture rate-limiting

- **Status:** draft — static analysis; **needs live PoC on an owned managed device**
- **Asset:** Linewize Connect extension v4.0.5 — in-scope focus area: "Bypass of Screen View."
- **Weakness type:** Business-logic / monitoring evasion (stale data presented as live)
- **Severity (CVSS v3.1):** provisional **~3.7 Low** — `AV:L/AC:H/PR:L/UI:N/S:U/C:L/I:L/A:N`
- **URL of vulnerability:** N/A (client-side extension logic)
- **Discovered:** 2026-09-24

## Summary

Screen View captures the active tab in the privileged background worker via
`chrome.tabs.captureVisibleTab` — a page cannot directly veto it, so the control
is fundamentally stronger than the Content Blocked overlay. However, when
captures hit Chrome's `MAX_CAPTURE_VISIBLE_TAB_CALLS_PER_SECOND` limit
(~2 calls/sec), the code **falls back to a cached screenshot up to 30 seconds old**
(`Wi.getPNG(n.id, 3e4)`) and still reports it. A student who can drive the
capture trigger into the rate limit could make the teacher's live view show a
frozen, innocuous frame while doing something else.

## Where it lives (shipped code)

- `background/background.bundle.js` → `Wi.captureActiveTab`:
  - Primary: `chrome.tabs.captureVisibleTab(e,{format:"png"})`.
  - On `MAX_CAPTURE_VISIBLE_TAB_CALLS_PER_SECOND`: `i = yield Wi.getPNG(n.id, 3e4)`
    — serves a cached PNG whose max age is 30 s; only if that's also missing does
    it surface `SCREEN_CAPTURE_RATE_LIMITED`.
  - Screenshot cache in IndexedDB (`screenshotStore` / `screenshotCache`).
- Capture is event-driven (`tabs.onActivated` / `onUpdated`) plus a periodic
  `ScreenshotUploadInterval` — so evasion windows depend on trigger cadence.

## Bypass angle (to confirm live)

Induce rapid capture triggers (e.g. rapid tab activation churn across windows)
to exhaust the per-second quota so the served frame is the ≤30 s cached one, then
change on-screen content within that window. Measure how stale the teacher view
actually gets and whether it is labelled as live.

## Impact

Monitoring integrity: teacher may see stale content as current. Lower severity —
no data of other users, no persistent bypass, bounded by capture cadence.

## Evidence

TODO: live capture on owned device showing served frame lag vs actual screen.

## Suggested remediation

Label or timestamp served frames; treat rate-limit fallback as "stale/unknown"
in the teacher UI rather than presenting a cached frame as live.

## Scope check

- [x] In-scope Linewize extension focus area
- [x] No DoS / no production degradation
- [x] Only devices we own · no other user's data
