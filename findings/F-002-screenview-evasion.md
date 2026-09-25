# [F-002] Screen View monitoring evasion — multiple client-observable blind spots

- **Status:** draft — static analysis complete; **needs live PoC on owned device/account**
- **Asset:** Linewize Connect extension v4.0.5 — in-scope focus area: "Bypass of Screen View."
- **Weakness type:** Monitoring/enforcement evasion; privacy feature repurposed as a universal blind (business-logic)
- **Severity (CVSS v3.1):** provisional **~4.3 Medium** (primary) — `AV:L/AC:L/PR:L/UI:N/S:U/C:L/I:L/A:N`
- **Discovered:** 2026-09 (static analysis of shipped bundle)

## Where it lives

Capture pipeline: `background.bundle.js` → `captureTabAndSend` (~L54473) →
`Wi.captureActiveTab` (`captureVisibleTab`). Live-view path: `getScreenshot`
(~L58668). Restricted logic: `getOpenRestrictedDomains` (L39182),
`restrictedDomains` from `screenshotsConfig.restrictedDomains` (L55833).

Screen View captures the **active tab** in the privileged background worker, so a
web page cannot directly veto it. Instead, several **client-observable states**
make the capture skip or upload a blank/placeholder frame.

## Primitive 1 — Restricted-domain placeholder swap (PRIMARY)

`getOpenRestrictedDomains(openTabUrls)` matches the config `restrictedDomains`
list against **every open tab** (not just the active one). In `captureTabAndSend`
(L54494/54532): if **any** open tab matches, `_ = true` →
`state = SCREEN_CAPTURE_RESTRICTED`, the frame is replaced with the bundled
`restricted-screenshot.txt` placeholder, and the real capture is **skipped**
(`S = false`) for the **entire** cycle. The live-view path checks the same
(`restrictedDomainFound`, L58691).

**Evasion:** park one restricted-listed domain in a pinned/background tab. Every
screenshot — scheduled *and* on-demand live view — is replaced by the bundled
placeholder (`restricted-screenshot.txt`, a JPEG reading **"Restricted
website/testing application detected, Live View disabled"** with a crossed-out
eye) while you do anything in another tab. A privacy feature becomes a universal
content-monitoring blind.

**Content-evasion, not stealth (accuracy for triage):** the placeholder openly
says "Live View disabled", so a watching admin *sees that the screen is hidden* —
they just can't see its contents. This defeats content monitoring, not the fact
that monitoring is being evaded. Report it as such; do not oversell as invisible.

The list is policy-pushed (`screenshotsConfig.restrictedDomains`) and lives in
device config. The placeholder text ("testing application detected") implies the
list includes security/testing tool domains as well as privacy categories — a
tester can discover which domains flip the placeholder **on their own screen**
empirically.

## Primitive 2 — Blank-frame starvation

The pipeline uploads an **empty** screen (or skips) on client-controllable states:

- `NoFocusedWindow` — `getLastFocusedWindow()` returns nothing (Chrome not the
  focused window) → empty upload (L54528).
- `InvalidTarget` / `SCREEN_CAPTURE_DISALLOWED` — active surface is DevTools or a
  `chrome://`/`edge://` page → empty/disallowed (L54520-54522).
- `TabNotLoaded` — active tab has a pending URL / never finishes loading → skip
  (L54520, L54548).

**Evasion:** keep the focused surface a browser-internal page (or Chrome
unfocused / a perpetually-loading tab) while real activity sits in an unfocused
window/tab → teacher receives blank frames.

## Primitive 3 — Rate-limit skip / stale live frame

Internal 10s same-URL limit (`InternalRateLimit`, L54524-54527) + Chrome's
~2/s `captureVisibleTab` cap. Scheduled path **skips** on
`SCREEN_CAPTURE_RATE_LIMITED` (L54550); live-view `captureActiveTab` falls back
to a **cached PNG up to 30s old** (`getPNG(id, 3e4)`). Rapidly churning tab
activations to exhaust the quota can freeze the live view on a stale frame.

## Primitive 4 — Coverage gaps (works-as-designed, worth reporting)

Capture interval only runs while `isClassroomEnabled()` **and** `hasActiveClass()`
(L54479-54488); otherwise it is **cancelled entirely**. Activity outside the
configured monitoring schedule is not captured. Companion-agent presence also
defers capture to the agent (`UploadRestrictedByCompanion`, L54574).

## Impact

Defeats/curtails the live monitoring that is this deployment's primary use.
Integrity of the monitoring record. No other user's data involved — tester's own
screen/account only.

## Evidence (TODO, owned device)

- P1: `.mp4` — restricted domain parked in a tab; teacher/live view shows the
  placeholder while other content is on screen. Note which domain triggered it.
- P2: blank frame while focus is on a `chrome://` page / unfocused window.
- P3: live-view frame lag vs actual screen under rapid tab churn.

## Suggested remediation

Scope the restricted check to the **active/visible** tab only, not all open tabs;
timestamp/label served frames and treat rate-limit fallback as "stale/unknown";
surface "blank/undeterminable" states to the teacher rather than a blank frame.

## Scope check

- [x] In-scope Linewize extension focus area (Screen View bypass)
- [x] No DoS / no production degradation
- [x] Only tester's own device/account · no other user's data
