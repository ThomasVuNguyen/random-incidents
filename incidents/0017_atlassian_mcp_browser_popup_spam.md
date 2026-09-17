# 0017 — Atlassian MCP browser popup spam on Antigravity IDE startup

## What happened

Every time Antigravity IDE was opened, the `atlassian-rovo-mcp` MCP server would start up and attempt to authenticate via OAuth through the browser. Because the Atlassian account credentials were stale or the OAuth app couldn't be identified (error: "We couldn't identify the app requesting access"), the browser would open multiple `id.atlassian.com` login tabs — sometimes tens of them — each showing an error page with a yellow warning triangle.

The root cause was that the `atlassian-rovo-mcp` server was configured in `~/.gemini/config/mcp_config.json` without a `"disabled": true` flag. It launched on every IDE session via a compat wrapper (`atlassian-mcp-compat.mjs`) that proxied to `mcp-remote` connecting to `https://mcp.atlassian.com/v1/mcp/authv2`. Since the Confluence data export (incident 0012) was already completed and the Atlassian subscription was being cancelled, there was no reason for this server to remain active.

## What I did to fix it

**First attempt** (failed): Added `"disabled": true` to the `atlassian-rovo-mcp` entry in `~/.gemini/config/mcp_config.json`. This did NOT work — popups continued because there were multiple cached MCP directories and running processes.

**Second attempt** (nuclear — worked): Completely removed everything Atlassian-related:

1. **Deleted the server entry** from `~/.gemini/config/mcp_config.json` entirely
2. **Deleted the compat wrapper script** `~/.gemini/config/atlassian-mcp-compat.mjs`
3. **Deleted all cached MCP tool schemas** across all 3 environments:
   - `~/.gemini/antigravity/mcp/atlassian-rovo-mcp/` + `atlassian-mcp-server/`
   - `~/.gemini/antigravity-cli/mcp/atlassian-rovo-mcp/` + `atlassian-mcp-server/`
   - `~/.gemini/antigravity-ide/mcp/atlassian-rovo-mcp/` + `atlassian-mcp-server/`
4. **Removed all Atlassian permission grants** from `~/.gemini/config/config.json` (both `globalPermissionGrants` and learned permissions)
5. **Force-killed 12 zombie Atlassian processes** (`mcp-remote` and `atlassian-mcp-compat.mjs` instances)

## What will prevent this from happening again

1. **The server is now disabled** — it won't launch unless explicitly re-enabled.
2. **General practice:** MCP servers that require browser-based OAuth should either have valid cached credentials or be disabled by default. Servers for cancelled/unused services should be disabled immediately after the service is decommissioned.
3. **The Confluence data export (incident 0012) was the last use of this integration** — with the Atlassian subscription being cancelled, there's no future need for this server.

---

**Signed:** Antigravity (Claude Opus 4.6 Thinking) — 2026-09-17
