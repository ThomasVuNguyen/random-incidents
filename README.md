# What are the incidents?

Sometimes shit happen (servers go down, keys get leaked, data gets corrupted, etc.) and I often let my Codex, Claude, Antigravity, OpenCode, Conductor, etc solve.

This is a place to log all of them.


# Summary

## 0001 — Mac mini and Coolify storage exhaustion — 2026-08-21

`cloud.comfyspace.tech` returned HTTP 500 after both the Mac mini Data volume and the production Lima VM filled. Codex cleared safe host caches, repaired the corrupted ext4 guest filesystem offline, pruned unused Docker images and build cache, trimmed the virtual disk, removed an abandoned application and its volumes, and completed an approved cleanup of obsolete local development data. Coolify returned healthy, with safe free-space headroom on both storage layers.

**Solved by:** Codex

**Report:** [incidents/0001_mac_mini_coolify_storage_exhaustion.md](incidents/0001_mac_mini_coolify_storage_exhaustion.md)

## 0002 — Mac mini power flicker left Coolify behind a 502 — 2026-08-26

Two power flickers rebooted the Mac mini. Cloudflare returned, but the production Lima VM stayed stopped after its startup job failed once and did not retry. Codex restarted the VM, verified healthy core services and public routes, then updated the startup job to retry failed launches every 30 seconds. After the owner completed the required local startup-security changes, a controlled reboot proved the unattended macOS, Lima, Coolify, Cloudflare, and public-route chain. Automatic restart after power failure is enabled.

**Solved by:** Codex

**Report:** [incidents/0002_mac_mini_power_flicker_coolify_502.md](incidents/0002_mac_mini_power_flicker_coolify_502.md)

## 0003 — Obsolete Data Cloud MCPs caused Antigravity errors — 2026-08-27

Antigravity's shared MCP catalog still contained four Google Data Cloud integrations after their extension proxy had been removed. Codex deleted only the orphaned `context`, `notebooks`, `visualization`, and `data-agent-kit` entries, preserved the rest of the catalog, and verified valid configuration, owner-only permissions, a clean MCP listing, and a completed authenticated model conversation without the missing-module error.

**Solved by:** Codex

**Report:** [incidents/0003_antigravity_obsolete_data_cloud_mcp.md](incidents/0003_antigravity_obsolete_data_cloud_mcp.md)

## 0004 — Antigravity MCP catalog audit and Zernio removal — 2026-08-27

Antigravity still displayed five red MCP rows after the Data Cloud cleanup. Codex established that MongoDB, Onshape, Penpot, and Pixels were disabled-but-visible, while Pencil remained enabled against an application that had been moved to Trash. Zernio was removed as requested, the other entries were preserved pending an owner decision, and a valid catalog plus a completed authenticated model response verified the change.

**Solved by:** Codex

**Report:** [incidents/0004_antigravity_mcp_catalog_audit_zernio_removal.md](incidents/0004_antigravity_mcp_catalog_audit_zernio_removal.md)

## 0005 — OpenCode installed but the active shell could not find it — 2026-08-27

OpenCode `1.18.23` installed correctly and added its executable directory to `.zshrc`, but the already-running Terminal session still had the old PATH. Codex verified the executable, confirmed the shell configuration, and proved that a fresh interactive zsh finds and runs OpenCode successfully. Reloading `.zshrc` fixes the existing tab; future Terminal windows work automatically.

**Solved by:** Codex

**Report:** [incidents/0005_opencode_path_not_loaded.md](incidents/0005_opencode_path_not_loaded.md)

## 0006 — macOS opened too many apps after login — 2026-08-31

macOS opened a mix of explicit Login Items, nested helpers, and previously open applications after restart. Codex removed every visible Login Item except Google Drive and Shottr, preserved Tailscale's login helper, disabled the other user and system startup services, and turned off session restoration. Mailspring and the remaining ChatGPT Atlas data were moved to a recoverable Trash folder; Clicky and the Atlas app bundle were already absent, so their stale startup records were cleared.

**Solved by:** Codex

**Report:** [incidents/0006_macos_startup_app_cleanup.md](incidents/0006_macos_startup_app_cleanup.md)

## 0007 — draw.beenex.org briefly returned an upstream 403 — 2026-09-01

`draw.beenex.org` briefly returned HTTP 403 even though the Coolify application, frontend, backend, Traefik, and host remained healthy and uninterrupted. Codex confirmed the rejected navigation never reached the application, isolated the event to the upstream Cloudflare/edge-to-Traefik path, avoided an unnecessary restart, and verified the origin plus the public root, asset, and API routes all returned HTTP 200 after the transient denial cleared.

**Solved by:** Codex

**Report:** [incidents/0007_draw_beenex_transient_upstream_403.md](incidents/0007_draw_beenex_transient_upstream_403.md)

## 0008 — Remote VM onboarding and wildcard DNS cutover — 2026-09-01

A newly provisioned remote VM was securely added to Coolify and assigned an existing Cloudflare wildcard route after an owner-approved origin cutover. Codex verified the host, registered and validated the server, recovered its initially missing Traefik listener, replaced the prior wildcard target, and proved public HTTP and HTTPS reached the new origin. A stale proxy status led to a failed queued restart and brief test-route 522; Codex detected it immediately, restored the Coolify-managed proxy, and reverified healthy host and public routing state.

**Solved by:** Codex

**Report:** [incidents/0008_remote_vm_coolify_wildcard_dns_cutover.md](incidents/0008_remote_vm_coolify_wildcard_dns_cutover.md)

## 0009 — No offsite database backups across Coolify infrastructure — 2026-09-05

A backup audit found that 4 of 5 PostgreSQL databases had zero backup schedules, and the one that did (peopledb-postgres) stored dumps only on the same server. No S3 or offsite storage was configured anywhere. Antigravity created a GCS bucket (`comfyspace-coolify-backup`) with S3-compatible HMAC credentials, added it as a Coolify S3 storage destination, and configured daily backup schedules with S3 upload for all 5 PostgreSQL databases — with 7-day local and 30-day offsite retention.

**Solved by:** Antigravity

**Report:** [incidents/0009_no_offsite_database_backups.md](incidents/0009_no_offsite_database_backups.md)
