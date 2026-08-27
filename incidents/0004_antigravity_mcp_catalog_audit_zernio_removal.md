# 0004 — Antigravity MCP catalog audit and Zernio removal

**Date:** 2026-08-27

**Service:** Antigravity CLI, Antigravity 2.0, and Antigravity IDE
**Status:** Resolved

![Five disconnected MCP entries under review while one obsolete service is removed](0004_antigravity_mcp_catalog_audit_zernio_removal.png)

## What happened

After the obsolete Google Data Cloud integrations were removed, Antigravity's MCP management screen still showed red rows for MongoDB, Onshape, Pencil, Penpot, and Pixels. Zernio was connected and exposed all 52 of its tools, but the owner no longer needed it and requested its removal.

The red rows did not all mean the same thing. Four were disabled entries that Antigravity continued to display as disconnected catalog items. Pencil was still enabled but could no longer start because its application had been moved to Trash.

## What I did to fix it

I removed only the `zernio` entry and its authorization header from the shared MCP catalog. I preserved all unrelated integrations and audited each red entry without changing it:

- **MongoDB:** disabled with a two-second timeout. Its configuration supplies an empty `MONGODB_URI`, while the installed server requires `MONGODB_MCP_CONNECTION_STRING`; the stored crash is therefore accurate. It is not currently running.
- **Onshape:** disabled with a two-second timeout. Its executable still exists, but prior live probing showed that Antigravity's nonstandard `server/discover` request is incompatible with this strict Python MCP server. It is not currently running.
- **Pencil:** enabled with no timeout, but `/Applications/Pencil.app` no longer exists. Older processes survive only because the executable is still memory-mapped from the app in Trash; a new Antigravity session cannot launch it from the configured path.
- **Penpot:** disabled with a two-second timeout. Penpot processes observed during the audit belong to Codex, not Antigravity, so they were left untouched.
- **Pixels:** disabled with a two-second timeout. Its configured `/mcp/sse` URL currently returns the application's HTML page rather than an MCP event stream. It is not currently running.

I also confirmed that the IDE's `agentPlugins/<name>` HTTP 404 messages are metadata lookups for custom MCP names, not proof that every named MCP connection failed; the same lookup appeared for healthy integrations.

Finally, I verified that the shared configuration remained valid JSON with owner-only `600` permissions, `agy mcp list` no longer returned Zernio, and a fresh authenticated conversation completed with `ZERNIO_REMOVAL_OK`.

## What should prevent it next time

- Delete unused disabled entries when a clean MCP management screen is preferred; disabling them can leave visible red rows.
- Keep disabled entries only when there is a realistic plan to repair or re-enable them.
- Validate required environment-variable names against the installed MCP package before enabling a server.
- Remove or update entries when their backing desktop application is uninstalled or moved.
- Probe remote MCP URLs for the expected protocol and content type rather than treating HTTP 200 alone as success.
- Verify configuration validity, catalog state, and a real model response after MCP changes.

— Codex, 2026-08-27
