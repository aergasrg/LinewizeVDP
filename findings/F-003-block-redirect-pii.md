# [F-003] Block redirect exposes employee & device identifiers in URL params

- **Status:** observation — **out-of-scope domain**; report as good-faith customer note + ask Qoria to confirm scope
- **Asset:** hosted block page `blocked.<region>.linewize.net/blocked` (redirect target). **NOTE: `linewize.net` is NOT in the VDP in-scope table** — observation only, no active testing.
- **Weakness type:** Sensitive information in URL / insufficient data minimisation (CWE-598)
- **Severity (CVSS v3.1):** provisional **~3.3 Low** — `AV:L/AC:L/PR:L/UI:R/S:U/C:L/I:N/A:N`
- **Discovered:** 2026-09 (observed incidentally in own browser history, as a customer)

> Real identifiers seen during observation are **redacted** here and must stay
> redacted in any submission/evidence. Placeholders used below.

## Summary

When a request is blocked, the browser is redirected (GET) to a block page whose
URL carries the target URL plus several identifiers as query params. GET params
persist in **browser history**, server **access logs**, and may leak via
**`Referer`**. Observed params:

```
https://blocked.<region>.linewize.net/blocked
  ?url=<blocked-url>
  &deviceid=<DEVICE_ID>            # hardware/asset device identifier
  &user=<EMPLOYEE_ID>             # employee/user number (plaintext)
  &rule=<base64>                  # e.g. "Block Url | Employee"
  &ruleid=<uuid>
  &path=
  &method=rule_match
  &cid=<base64(EMPLOYEE_ID + epoch_ms)>   # user id + timestamp
```

The block page (screenshot on file, redacted) also renders: Website, Path,
Policy Name, Rule Type, Application/Category, and **public IP**.

## Notes / verification done

- Reflected params (`url`, policy name) render **HTML-encoded** — **no XSS**
  (tested on own params only; this is the correct/secure behaviour, recorded so
  triage knows it was checked).
- `cid` decodes to `<EMPLOYEE_ID>` concatenated with an epoch-millis timestamp.
- `rule` is base64 of the human policy name.

## Impact

Low. Own-identity exposure into history/logs/Referer; aids correlation. No other
user's data accessed. Not a control bypass.

## Explicitly NOT tested (out of scope)

- Changing `user=` / `deviceid=` / `cid=` to **other** users' values (would be
  IDOR against another person's data on an out-of-scope host). **Do not.**
- Any injection/fuzzing of the block page or the server-side **bypass-code**
  validation (server-side auth control, out-of-scope domain).

## Bypass-code architecture (for the record)

The "Enter your bypass code to unblock" flow is **server-side**: the extension
receives `bypassCode` + `bypassExpiryTime` in the verdict/policy result and only
reports/honours them (see `background.bundle.js` policyResult handling). No
client-side code comparison exists to attack. The plaintext bypass code does
travel in the extension's uploaded activity telemetry — minor note only.

## Recommended action

Report as a good-faith **customer** observation (VDP accepts customer reports)
and **ask Qoria whether `*.linewize.net` is in scope**, since core product flows
depend on it but it's absent from the scope table. Suggest POST/opaque token
instead of identifiers in GET params.
