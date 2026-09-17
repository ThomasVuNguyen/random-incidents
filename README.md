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

## 0010 — Antigravity & Antigravity IDE chat history cleanse and archival — 2026-09-06

Antigravity Desktop and Antigravity IDE had accumulated 1,787 conversation session files, 836 brain workspaces, and over 4.7 GB of local session histories and workspace caches. Antigravity safely created complete compressed archives of all active chat sessions, brain workspaces, annotations, summaries, and workspace states to `~/.gemini/backups/`, verified all backups, and completed a full cleanse of conversation histories and caches across both environments without impacting active CLI operations, system settings, or MCP configurations.

**Solved by:** antigravity gemini 3.8 flash

**Report:** [incidents/0010_antigravity_and_ide_chat_history_cleanse.md](incidents/0010_antigravity_and_ide_chat_history_cleanse.md)

## 0011 — BillulloAgentic GCP billing cost investigation — 2026-09-09

The BillulloAgentic billing account was unusually expensive over the last 60 days. Investigation across 5 active GCP projects identified root causes and all were remediated: (1) `wia` Cloud Run minScale set from 1→0 after repo analysis confirmed no need for always-on (~$45/mo saved), (2) 640 stale Docker images deleted from Artifact Registry across 11 repos (~$6/mo saved), (3) Cloud Scheduler frequencies reduced — notification outbox from 5min→30min, BeeSecurity scan from 15min→weekly after repo analysis confirmed static posture scanning doesn't need sub-hourly runs (~$21/mo saved), (4) Vertex AI audited across all projects — kept enabled per owner decision, (5) stale bloom/miles services confirmed at $0 cost (minScale=0, zero traffic). **Total savings: ~$72/month (~$860/year).** Expected idle cost: ~$2-5/month.

**Solved by:** Antigravity

**Report:** [incidents/0011_billulloagentic_billing_cost_investigation.md](incidents/0011_billulloagentic_billing_cost_investigation.md)

## 0012 — Confluence full data export before subscription cancellation — 2026-09-10

BillulloNex needed to export all Confluence data before canceling their Atlassian subscription. Antigravity authenticated via the Atlassian Rovo MCP, enumerated all 25 spaces (22 current + 3 archived), and bulk-fetched 69 pages with full markdown content. An initial parallel subagent strategy failed because MCP permission prompts don't propagate to subagents, so the export was completed directly via bulk API calls. All pages were saved as both Markdown (with YAML frontmatter) and JSON (with full metadata) to a structured local export directory, with per-space manifests and a global index.

**Solved by:** Antigravity (Claude Opus 4.6 Thinking)

**Report:** [incidents/0012_confluence_full_data_export.md](incidents/0012_confluence_full_data_export.md)

## 0013 — Compute Engine RDP session establishment timeout — 2026-09-11

Remote Desktop connections to `platinum-it-windows-6cpu-24gb-prod` reached the VM and authenticated but timed out while Windows created the session. Codex isolated the failure to a wedged Remote Desktop Services stack, attempted a targeted service restart, and rebooted the VM after `UmRdpService` became stuck in `STOP_PENDING`. Successful `thomas` and `ptuser06` logons, multiple active RDP sessions, and healthy RDP services verified the recovery.

**Solved by:** Codex

**Report:** [incidents/0013_compute_engine_rdp_session_timeout.md](incidents/0013_compute_engine_rdp_session_timeout.md)

## 0014 — `cloud.comfyspace.tech` intermittently returned Cloudflare Tunnel 1033 — 2026-09-12

Cloudflare intermittently returned HTTP 530 / Tunnel error 1033 for the Coolify control-plane URL. Public and direct-origin probes failed together briefly, then both recovered without a restart; the tunnel connector was active again afterward. The incident was isolated to a transient Mac mini/tunnel reachability flap, with no evidence of a Lenovo workload or DNS failure.

**Solved by:** Codex (automatic recovery observed; recurrence prevention remains)

**Report:** [incidents/0014_cloudflare_tunnel_1033_flap.md](incidents/0014_cloudflare_tunnel_1033_flap.md)

## 0015 — Large ingestion exhausted VM memory headroom — 2026-09-12

A large ingestion job left the shared `beenex.cloud` VM with only a few hundred MiB of available RAM and no swap. Severe memory reclaim produced high I/O wait and dozens of blocked processes. Codex added a persistent 16 GiB low-swappiness safety buffer, then permanently removed the retired Twenty stack after owner approval and verified that the separately deleted Teable stack left no Coolify or host residue. Memory use fell from about 13 GiB to 4 GiB, with about 10 GiB available and unaffected routes healthy.

**Solved by:** Codex

**Report:** [incidents/0015_ingestion_memory_pressure_swap_safety_net.md](incidents/0015_ingestion_memory_pressure_swap_safety_net.md)

## 0016 — Rybbit Globe showed a CARTO API-key watermark — 2026-09-14

Rybbit 2.6.0's 2D Globe depended on a CARTO raster-tile endpoint that began watermarking unauthenticated requests, even though the BeeNex deployment had not changed. Codex confirmed the upstream cause, force-redeployed the existing Coolify stack to Rybbit 2.9.1, and verified the health endpoint, preserved analytics data, OpenFreeMap attribution, and a watermark-free live Globe.

**Solved by:** Codex

**Report:** [incidents/0016_rybbit_carto_map_api_key_watermark.md](incidents/0016_rybbit_carto_map_api_key_watermark.md)

## 0017 — Atlassian MCP browser popup spam on Antigravity IDE startup — 2026-09-17

The `atlassian-rovo-mcp` MCP server was still enabled in `~/.gemini/config/mcp_config.json` after the Atlassian subscription was being cancelled. Every IDE launch triggered OAuth authentication attempts that opened tens of `id.atlassian.com` browser tabs, all showing "We couldn't identify the app requesting access" errors. Antigravity disabled the server by adding `"disabled": true` to its config entry.

**Solved by:** Antigravity (Claude Opus 4.6 Thinking)

**Report:** [incidents/0017_atlassian_mcp_browser_popup_spam.md](incidents/0017_atlassian_mcp_browser_popup_spam.md)
