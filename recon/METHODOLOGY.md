# Recon & testing methodology

Everything here stays inside `../SCOPE.md`. Passive/static first, active only on
accounts we own, never anything that degrades production.

## 0. Ground rules (repeat until reflex)

- In-scope domains only. No DoS. No scanners that hammer. Accounts we own only.
- Reach another user's data → **stop and report.** Extract nothing.

## 1. Passive recon (safe, do first)

- Confirm live in-scope hosts and their `security.txt` (RFC 9116) for contacts.
- Map the surface we already have legit access to (our org's Classwize / School
  Manager / Monitor consoles on `*.familyzone.io`).
- Note the API hosts the consoles call (`*.qoria*`) from browser dev tools —
  observation, not fuzzing.

## 2. Extension static analysis (Linewize Connect)

- Read `manifest.json`: permissions, content_scripts, host matches, web-
  accessible resources, externally_connectable.
- Trace how the **Content Blocked** screen and **Screen View** are enforced
  client-side — the two invited bypass targets.
- Look for enforcement that trusts client state, removable DOM overlays, race
  windows before injection, messaging that can be spoofed.

## 3. Authenticated testing (accounts we own)

Focus-area oriented:

- **Access control / IDOR:** swap object IDs between two accounts we control;
  does role separation hold (student vs teacher vs admin)?
- **Privilege escalation:** can a lower role reach higher-role functions/APIs?
- **Broken auth:** session handling, token scope, password/OTP flows.
- **Business logic:** flows that expose data or functionality out of band.
- **Injection / XSS:** only where it reaches session/sensitive data; PoC, no
  weaponisation.

## 4. Discipline

- One vuln → one finding file (`findings/F-00X-*.md`). Chain only for impact.
- Redact real data in all evidence.
- Log each active test (what, when, which account) so we can prove good faith.

## Static-analysis notes (extension v4.0.5)

Source maps had no `sourcesContent`, but exposed original module names; bundles
beautified for reading. Architecture confirmed:

- **Content Blocked** = network `webRequest.onBeforeRequest` block **only when
  verdict is cached**; uncached → request allowed + async tab redirect + a
  removable content-script white overlay with a 5.5 s fail-open timer and a
  naive keyword fallback. → `F-001`.
- **Screen View** = privileged `captureVisibleTab` in the background (page
  cannot veto), but rate-limit fallback serves a ≤30 s cached frame. → `F-002`.

Minor observations (not yet findings):

- `manifest.json` `exclude_matches: https://*.linewize.net/*` — filter/categoriser
  don't run on that host; plus a runtime `CHECK_DISABLE_CONTENT_SCRIPT`.
- `web_accessible_resources` exposes `*.js.map` to all origins (minor info leak,
  aids exactly this kind of analysis).
- `isGoogleMapsVerdictBypass` — a hardcoded verdict bypass path worth examining.

## Test log

| Date | Target | Action | Account | Notes |
|---|---|---|---|---|
| 2026-09-24 | ext v4.0.5 bundle | static analysis only (no live traffic) | n/a | mapped enforcement → F-001, F-002 |
