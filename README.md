# LinewizeVDP

i hate my life — but now with a CVSS tracker

Workspace for good-faith security research under the **Qoria Vulnerability
Disclosure Policy** (covers Linewize / Family Zone / Qustodio / CipaFilter and
friends). Goal: protect student and staff data, report findings the right way,
stay firmly inside the safe harbour.

## Layout

- [`SCOPE.md`](SCOPE.md) — in-scope domains, the extension carve-out, hard walls,
  program rules. **Read before touching anything.**
- [`recon/METHODOLOGY.md`](recon/METHODOLOGY.md) — how we test, safely, plus the
  active-test log.
- [`findings/`](findings/) — one file per vulnerability, from
  [`findings/TEMPLATE.md`](findings/TEMPLATE.md), mirroring Qoria's submission form.

## The one rule that matters most

If testing ever surfaces another user's data: **stop immediately and report.**
Extract nothing, share nothing. This is minors' and staff data — that line is the
entire reason the policy protects us.
