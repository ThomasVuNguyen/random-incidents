# 0016 — Rybbit Globe showed a CARTO API-key watermark

**Date:** 2026-09-14

**Service:** `rybbit.beenex.org`  
**Status:** Resolved

![Rybbit map recovering from a third-party tile-provider warning](0016_rybbit_carto_map_api_key_watermark.png)

## What happened

The 2D Globe view in the self-hosted Rybbit dashboard suddenly displayed repeated `API KEY REQUIRED` watermarks across its world map. Analytics data and the rest of the dashboard still loaded normally.

This was an upstream dependency change, not a leaked or deleted BeeNex key. Rybbit 2.6.0 hardcoded CARTO's unauthenticated `basemaps.cartocdn.com` raster-tile endpoint for the 2D map. CARTO began watermarking those requests when it started requiring an API key, so the visual failure appeared even though no BeeNex configuration or deployment had changed.

Rybbit had already fixed the issue upstream in [change #1145](https://github.com/rybbit-io/rybbit/pull/1145) by replacing the CARTO basemap with OpenFreeMap. The fix was included in [Rybbit 2.9.0](https://github.com/rybbit-io/rybbit/releases/tag/v2.9.0) and remains in 2.9.1.

The Coolify application used `IMAGE_TAG=latest`, but its last deployment was August 2. The running containers therefore still held the older 2.6.0 images even though newer `latest` images were available.

## What I did to fix it

- Compared the exact 2D Globe implementation in Rybbit 2.6.0 and 2.9.1. Version 2.6.0 used the CARTO raster endpoint; 2.9.1 uses `https://tiles.openfreemap.org/styles/dark`.
- Confirmed that the published Rybbit `latest` client and backend image digests matched the 2.9.1 image digests.
- Forced a normal Coolify redeployment of the existing Rybbit Docker Compose application so it pulled the current images and recreated the application containers.
- Preserved the existing PostgreSQL and ClickHouse named volumes; no analytics data or site configuration was deleted.
- Waited for Coolify deployment `gbjcjephbe6y7pjlyqovyefh` to finish successfully in 85 seconds.
- Verified `https://rybbit.beenex.org/api/health` returned HTTP 200 with `OK`, and `/api/version` reported `2.9.1`.
- Opened the authenticated BeeNex Globe and confirmed that the map attribution now identifies OpenFreeMap, OpenMapTiles, and OpenStreetMap; the CARTO/API-key watermark is gone and existing session data still appears.

An unrelated controlled/uncontrolled select warning remains in the browser console. It did not affect the Globe fix or map rendering.

## What should prevent it next time

- Treat mutable image tags such as `latest` as update channels, not automatic updates: a running container keeps its existing image until a pull and redeploy occurs.
- Review Rybbit release notices and redeploy periodically so upstream dependency fixes are actually applied.
- Consider pinning `IMAGE_TAG` to an explicit tested version for predictable upgrades, then advance it deliberately after release review.
- Keep a small post-upgrade probe that checks the health endpoint, loads an authenticated Globe, and asserts that the expected map-provider attribution appears without provider-error watermarks.
- Continue preserving PostgreSQL and ClickHouse volumes across application-container upgrades, and verify an existing site and session after each upgrade.

— Codex, 2026-09-14
