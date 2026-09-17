# 0014 — `cloud.comfyspace.tech` intermittently returned Cloudflare Tunnel 1033

**Date:** 2026-09-12

**Service:** `cloud.comfyspace.tech`
**Status:** Recovered; recurrence cause remains under investigation

![Cloudflare tunnel recovering after a brief connector and origin reachability flap](0014_cloudflare_tunnel_1033_flap.png)

## What happened

The Coolify control-plane URL showed Cloudflare Error 1033, surfaced as HTTP 530. The public DNS and TLS edge were still reachable, but Cloudflare temporarily could not reach the tunnel connector. The attached browser screenshot captured the same condition at approximately 07:03:50 UTC.

The failure was intermittent. Five consecutive public probes between 07:09:32 and 07:09:40 UTC returned HTTP 530 with the response body `error code: 1033`. During that same window, direct requests to the Mac mini origin timed out. Tailscale still reported the Mac mini online, so this was a short host/network or connector reachability flap rather than a permanent DNS or application configuration failure.

The route recovered without a restart. At 07:10:19 UTC the public endpoint returned the expected HTTP 302 redirect to `/login`; the direct origin returned 302 as well. A subsequent stability pass produced six consecutive public 302 responses and six matching direct-origin 302 responses. The `comfyspace-mac-mini` tunnel showed an active Mac connector after recovery.

## What I did to debug it

- Reproduced the browser symptom with `curl`; the response was HTTP 530 with body `error code: 1033`.
- Confirmed DNS still resolved through Cloudflare and captured fresh Ray IDs without exposing them in this report.
- Compared the public route with the direct Mac mini origin on port 8000.
- Verified the Mac mini and Lenovo remained visible through Tailscale, and checked the exact Cloudflare tunnel inventory from the reachable Lenovo host.
- Confirmed Lenovo's Coolify proxy and workloads were healthy and did not restart or change them.
- Confirmed the Mac mini tunnel connector was active again after the transient failure.
- Did not force a restart while the route had recovered; the available SSH account also lacked an accepted host key/auth path for administering the Mac mini.

## What should prevent it next time

- Add an external health check for `cloud.comfyspace.tech` that records HTTP status, response body, Ray ID, and timestamps when 1033/530 occurs.
- Add a second direct-origin/Tailscale probe so the next event distinguishes a Cloudflare connector failure from a Mac mini network or host pause immediately.
- Restore a verified administrative path to the Mac mini, including managed host-key trust and a tested Tailscale SSH/user authorization path.
- When access is available, inspect the `com.cloudflare.cloudflared` launchd job for `KeepAlive`, reconnect logs, and macOS sleep/network events; verify it survives a controlled connector restart.
- Plan a maintenance update for the Mac mini connector, which is behind the current cloudflared release, but do not treat version drift alone as the proven cause of this event.

— Codex, 2026-09-12
