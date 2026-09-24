# Qoria / Linewize VDP — Scope

Source: Qoria Vulnerability Disclosure Policy (qoria.com). Testing is **only**
permitted within the scope defined here. When in doubt, treat it as out of scope
and ask.

## In scope (domains)

| Product | Domains |
|---|---|
| Qustodio | `*.qustodio.com`, `*.qustodio.net` |
| **School Manager & Classroom Manager** (Linewize) | `*.familyzone.io` |
| Smoothwall Firewall | N/A |
| Record Manager | `app.safeguard.software` |
| CipaFilter | `*.cipafilter.com` |
| Educator Impact | `*.educatorimpact.com` |
| Family Zone | `*.familyzone.com` |
| Qoria (backend / APIs) | `*.qoria.com`, `*.qoriaapis.cloud`, `*.qoria.cloud` |

> **Where Linewize lives:** the brand has no own row. Its dashboards (Classwize,
> Monitor, School Manager) ride on `*.familyzone.io`; the backend/APIs sit under
> the `*.qoria.*` block. Those are the assets our org logs into daily.

## The Linewize Connect extension — read twice

The extension (`Linewize Connect`) is listed **out of scope** for general
testing, **except** two behaviours that the policy's Focus Areas explicitly
invite:

- Bypass of the **"Content Blocked"** screen on blocked websites.
- Bypass of **Screen View**.

Static analysis of the extension's own client-side code, on a machine we own, is
fair game and is the safe route to both of the above. Do not use it to reach
other users' data.

## Out of scope — hard walls

- `*.cybersafetyhub.*` — fully off-limits.
- **Denial of service** or any degradation of production services (no hammering
  scanners, no load/stress testing).
- **Social engineering** of customers or employees (phishing, vishing, smishing).
- Anything not on the in-scope list above.

## Program rules that shape how we test

- Detailed reports, reproducible steps. One vuln per report (chain only if
  needed for impact). Same API endpoint / different params → one report.
- **Only interact with accounts we own** or with the holder's explicit permission.
- **If we reach another user's data: stop immediately and report.** Extract
  nothing, share nothing. (This is kids' and staff data — the line is the point.)
- Don't leave any system more vulnerable than we found it.
- No extortion. Be respectful. Don't discuss findings outside the program
  without Qoria's consent.

## Focus areas (what they want)

Broken access control · privilege escalation · PoC access to sensitive info ·
RCE · XSS reaching session/sensitive data · SQLi reaching sensitive
data/functionality · broken authentication · IDOR · business-logic flaws
reaching sensitive data/functionality · (extension only) the two bypasses above.

## Submission form fields (mirror these in every finding)

Submission Title · Asset · Weakness Type · Severity (CVSSv3.1 preferred) ·
Description · URL of vulnerability · Attachments (.txt/.jpeg/.mp4) · Email
