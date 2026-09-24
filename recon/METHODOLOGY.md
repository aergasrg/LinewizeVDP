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

## Test log

| Date | Target | Action | Account | Notes |
|---|---|---|---|---|
| | | | | |
