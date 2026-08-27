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
