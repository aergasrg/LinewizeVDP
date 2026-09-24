# Linewize Connect — data flow & identity model (v4.0.5)

From static analysis of the shipped bundle. This is the attack-surface map.

## How it identifies the device / user (three overlapping signals)

1. **Google account** — OAuth (`identity`, `userinfo.email`); the soft identity
   (the signed-in Workspace email).
2. **Hardware / MDM identity (ChromeOS only, un-spoofable from userland)** — via
   `enterprise.deviceAttributes`: `getDeviceSerialNumber`, `getDirectoryDeviceId`,
   `getDeviceAssetId`, `getDeviceHostname`, `getDeviceAnnotatedLocation`
   (`background.js:61093`). Binds the device to the school's directory.
3. **Network identity** — `whoami.linewize.net` polled ~every 3 min
   (`WhoamiLoginInterval`); returns `provider_username` / `network_identity_provided`.
   If the network says you're a different user, config is re-pulled.

`applianceId` = the school's Family Zone appliance/tenant id; `userIdentifier` =
the user. Both are attached to Sentry, activity uploads, etc.

## What it talks to (hostnames seen in the bundle)

| Purpose | Host (suffix from config) | Notes |
|---|---|---|
| Policy/config | `configuration-gw.*` | classroom configs, teacher ids, rules, intervals |
| Auth/login | `login.*`, `chromelogin.linewize.net`, `securetoken.google.com`, Firebase | 10-min re-login loop |
| Identity resolve | `whoami.linewize.net` | network-identity check |
| Filtering verdicts | `api.*` | per-URL verdict (cached → sync block; uncached → async, see F-001) |
| Activity upload | school_manager `WindowUpdateRequest` (protobuf) | active window, all tabs, background tabs, screenshot, "restricted" flag, teacherIds |
| Live View | realtime push (**Ably**, in the offscreen doc) | teacher-triggered `CaptureTabAndSend` |
| Network-safety probe | `fzbox.tools` (`sit.`/`stg.`) | `getInsideCloudSafeNetwork()` — behavior changes on/off the school box |
| Telemetry | Sentry (`o18924.ingest.sentry.io`), `stats-xlb.*` | error + perf |

## Behavioral model

- **Filtering:** background `webRequest.onBeforeRequest` → `getVerdict`. Cached
  verdict = synchronous network block. Uncached = request allowed through +
  async tab redirect + content-script overlay (this is the F-001 hole).
- **Monitoring:** background captures the active tab (`captureVisibleTab`) on tab
  events + interval, uploads `WindowUpdate` protobuf; Live View pulls on demand
  over the Ably channel (F-002 touches the capture cache).
- **Network awareness:** `fzbox.tools` probe decides "safe network" — if
  enforcement relaxes when it believes it's on a safe network, spoofing that
  signal is a candidate bypass (**lead: trace `getInsideCloudSafeNetwork`**).

## Scope note (important)

The extension relies on **`linewize.net`** (`whoami`, `chromelogin`), which is
**NOT in the VDP in-scope domain table** (`*.familyzone.io`, `*.qoria.*`,
`*.qustodio.*`, etc. are). Observing the extension contact `linewize.net` on an
owned device is fine; **do not actively test/fuzz `*.linewize.net` endpoints** —
ask Qoria to confirm scope first.
