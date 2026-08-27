# 0003 — Obsolete Data Cloud MCPs caused Antigravity errors

**Date:** 2026-08-27

**Service:** Antigravity CLI, Antigravity 2.0, and Antigravity IDE
**Status:** Resolved

![Three Antigravity clients continuing through a cleaned MCP configuration](0003_antigravity_obsolete_data_cloud_mcp.png)

## What happened

Antigravity showed a persistent yellow MCP error. Its shared MCP catalog contained four enabled Google Data Cloud integrations—`context`, `notebooks`, `visualization`, and `data-agent-kit`—that all launched the same version-specific JavaScript proxy inside the Antigravity IDE extensions directory.

The Data Cloud extension and proxy file were no longer installed. Node therefore returned `MODULE_NOT_FOUND`, the MCP connection closed during initialization, and Antigravity surfaced the failure even though its authentication and model-generation service remained healthy.

## What I did to fix it

I audited the CLI, Antigravity 2.0, and Antigravity IDE independently and confirmed that the four obsolete integrations were consumed from the shared catalog at `/Users/thomasthemaker/.gemini/config/mcp_config.json`.

I removed only `context`, `notebooks`, `visualization`, and `data-agent-kit`. All unrelated MCP integrations were preserved.

I then verified that:

- the shared configuration remained valid JSON;
- its owner-only `600` permissions were preserved;
- no Data Cloud extension or proxy references remained;
- `agy mcp list` no longer returned any of the four obsolete integrations; and
- a fresh authenticated CLI conversation completed and stored the exact response `DATACLOUD_REMOVAL_OK`, without a Data Cloud proxy, missing-module, or MCP initialization error in its verification log.

## What should prevent it next time

- Do not retain MCP commands that point into a removed extension directory.
- Avoid absolute paths containing extension version numbers when a stable executable or managed integration is available.
- After extension upgrades or removals, compare enabled MCP entries against the files that actually exist.
- Keep startup timeouts on local MCP integrations so one unavailable server cannot delay the whole client indefinitely.
- Verify both the MCP catalog and a real model conversation after MCP changes; a valid configuration file alone is not enough.

— Codex, 2026-08-27
