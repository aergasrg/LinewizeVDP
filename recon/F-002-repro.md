# F-002 live repro plan — Screen View evasion

Own device + own account only. You are testing whether **your own** screen gets
captured — no other user's data is ever involved. Confirm a monitoring class is
**active** first (otherwise the interval is cancelled and nothing captures — see
Primitive 4). Ideally verify against what the teacher/admin console actually
shows, or the uploaded frames, with permission.

## Prep — is capture even running right now?

Screen View only fires when `isClassroomEnabled()` **and** `hasActiveClass()`.
So pick a window when you know monitoring is on. If nothing ever uploads, that's
Primitive 4 (schedule/coverage gap), itself worth noting.

## Test 1 — Restricted-domain placeholder swap (the big one)

Goal: make every screenshot become the "restricted" placeholder by keeping a
restricted-listed domain open somewhere.

1. **Discover the list empirically:** open candidate privacy domains one at a time
   in a background tab (major banks, health portals, gov/tax sites — categories
   orgs typically mark "do not screenshot"). After each, trigger a capture (switch
   tabs / wait an interval) and check whether the frame flips to the restricted
   placeholder.
2. Once one triggers it, **pin it in a background tab** and browse normally in
   another tab.
3. Confirm the teacher/live view shows the placeholder, not your real screen,
   the whole time. Record it. **Note exactly which domain triggered it.**

Because the check scans *all* open tabs, the restricted tab doesn't even need to
be active. That's the finding.

## Test 2 — Blank-frame starvation

- **Unfocused Chrome:** move focus to another OS window/app; captures log
  `NoFocusedWindow` → blank. (Easier on desktop Chrome than single-window CrOS.)
- **Browser-internal active tab:** keep a `chrome://settings` / DevTools surface
  focused while real content sits in another window/tab → `InvalidTarget` blank.
- **Perpetually-loading tab:** focus a tab that never finishes loading (pending
  URL) → `TabNotLoaded` skip.

Record which states produce blank/skipped uploads.

## Test 3 — Stale live frame under rate limit

Rapidly churn tab activations (many quick switches) to exhaust
`captureVisibleTab` (~2/s) + trip the internal 10s same-URL limit. Watch whether
the **live view** freezes on a ≤30s cached frame while your real screen changes.

## Capture for submission

Per primitive: short `.mp4` of real screen vs what the monitor received, the
triggering condition, and confirmation it's your own account only.
