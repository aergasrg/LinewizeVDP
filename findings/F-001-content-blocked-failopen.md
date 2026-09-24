# [F-001] Content Blocked screen fails open for uncached, non-sexual URLs

- **Status:** draft — static analysis complete, **needs live PoC on an owned managed device before submission**
- **Asset:** Linewize Connect extension (`Linewize Connect` v4.0.5) — in-scope focus area: "Bypass of the Content Blocked screen on blocked websites."
- **Weakness type:** Improper enforcement / fail-open security control (client-side bypass)
- **Severity (CVSS v3.1):** provisional **~5.3 Medium** — `AV:L/AC:L/PR:L/UI:N/S:U/C:L/I:L/A:N`
  (Qoria applies their own CVSS + contextual scoring; this is a starting point.)
- **URL of vulnerability:** N/A (client-side extension logic)
- **Discovered:** 2026-09-24 (static analysis of shipped bundle)

## Summary

For any URL whose filtering verdict is **not already cached**, the extension
does **not** block the network request synchronously. Enforcement instead falls
back to (a) an after-the-fact tab redirect and (b) a client-side "hide the page"
overlay in the content script that **auto-reveals the page after 5.5 seconds** if
a verdict has not arrived — gated only by a trivial keyword filter that looks for
sexual terms. Any blocked page in a **non-sexual** category (gambling, drugs,
weapons, self-harm, proxies/VPNs, gaming, social, streaming, etc.) is therefore
fully loaded in the DOM and becomes visible if the verdict is delayed or
prevented.

## Where it lives (shipped code)

- `background/background.bundle.js` → `onBeforeRequestFilter`. When
  `getVerdict()` returns a `Promise` (uncached URL), the handler takes the
  `"async"` branch and **returns `{}`** — i.e. the network request is *allowed*.
  Synchronous `{cancel:true}` / `{redirectUrl}` blocking only happens when a
  verdict is already cached. Enforcement for uncached URLs is deferred to an
  async `redirectTabToBlockedPage(...)` and the content-script overlay.
- `content_scripts/filter.bundle.js`:
  - The "Content Blocked" state during a pending verdict is a
    `position:fixed; z-index:2147483647` white `<div id="linewize-protect">`
    appended to `document.body`. The real page is fully loaded underneath it.
  - `getVerdictAndUnhide()` arms `whitePageRemoveMs = 5500`; on timeout it runs
    `documentLoadFallbackCheck()` → `runFallbackChecks()` and **unhides the page**
    unless the fallback filter trips.
  - Fallback filter (`checkPage`): `title` matches `/porn/i`, or body text
    contains one of `age-restricted`, `porn`, `sex`, `adult`, `fuck`, plus a few
    adult-disclaimer phrases. Keyword list: `["porn","sex","adult"]`. Anything
    outside this list passes → page revealed.

## Bypass primitives (each to be confirmed live)

1. **Verdict stall / fail-open (primary).** Prevent or delay delivery of the
   async verdict to the content script (candidates: MV3 service-worker idle
   eviction on first navigation, saturating the runtime message path, offline
   toggling — `CHECK_INTERNET_CONNECTION` failure resolves to ALLOW). After 5.5 s
   the non-sexual blocked page auto-reveals.
2. **Overlay removal.** The block is a removable DOM node. `remove()` on
   `#linewize-protect` (devtools console, reader mode, view-source, or a
   competing userscript) exposes the page, which was never prevented from
   loading. (Reduced impact where managed policy disables devtools.)
3. **`maxPageHideAttempts` exhaustion.** The hider gives up after 5 attempts
   (`blockCounter > maxPageHideAttempts`) — worth testing whether repeated
   hide/unhide cycling permanently disables the overlay.

## Impact

Defeats the core CIPA-compliance promise of the product for a broad set of
categories on first visit / after cache eviction. No other user's data is
involved — this is a control-integrity bypass on the tester's own device.

**Reachable without DevTools.** Confirmed candidate methods work on a fully
locked-down managed device (DevTools / `javascript:` / bookmarklets disabled):
the extension's own `@media print { #linewize-protect { display:none } }` rule
reveals the loaded page in Print Preview, and the OS Wi-Fi toggle starves the
verdict into the 5.5s fail-open. A control a normal student can bypass with the
print button raises real-world severity above the base CVSS. See
`../recon/F-001-repro.md` Tests P / W / U.

## Evidence

TODO: capture on an owned, managed device — screen recording (.mp4) of a
non-sexual blocked URL loading through, plus the relevant `console.log`
lines the extension already emits ("Hiding page…", "Verdict received…",
"showing page again"). **Redact org/appliance identifiers.**

## Suggested remediation

Fail **closed** on verdict timeout; enforce blocking at the network layer
(`declarativeNetRequest` / synchronous `cancel`) rather than a removable overlay;
do not rely on a client-side keyword list as the safety net.

## Scope check

- [x] Asset is the in-scope Linewize extension focus area
- [x] No DoS / no production degradation (local, own device)
- [x] Only accounts/devices we own
- [x] No other user's data touched
