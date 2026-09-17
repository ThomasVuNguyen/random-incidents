# 0017 — Atlassian MCP browser popup spam on Antigravity IDE startup

## What happened

Every time Antigravity IDE was opened, the `atlassian-rovo-mcp` MCP server would start up and attempt to authenticate via OAuth through the browser. Because the Atlassian account credentials were stale or the OAuth app couldn't be identified (error: "We couldn't identify the app requesting access"), the browser would open multiple `id.atlassian.com` login tabs — sometimes tens of them — each showing an error page with a yellow warning triangle.

The root cause was that the `atlassian-rovo-mcp` server was configured in `~/.gemini/config/mcp_config.json` without a `"disabled": true` flag. It launched on every IDE session via a compat wrapper (`atlassian-mcp-compat.mjs`) that proxied to `mcp-remote` connecting to `https://mcp.atlassian.com/v1/mcp/authv2`. Since the Confluence data export (incident 0012) was already completed and the Atlassian subscription was being cancelled, there was no reason for this server to remain active.

## What I did to fix it

Added `"disabled": true` to the `atlassian-rovo-mcp` entry in `~/.gemini/config/mcp_config.json`. This prevents the MCP server from launching on IDE startup while preserving the configuration in case it's ever needed again.

**File changed:** `~/.gemini/config/mcp_config.json`

```diff
     "atlassian-rovo-mcp": {
+      "disabled": true,
       "command": "/Users/thomasthemaker/.nvm/versions/node/v24.14.1/bin/node",
       "args": [
         "/Users/thomasthemaker/.gemini/config/atlassian-mcp-compat.mjs"
       ]
     },
```

## What will prevent this from happening again

1. **The server is now disabled** — it won't launch unless explicitly re-enabled.
2. **General practice:** MCP servers that require browser-based OAuth should either have valid cached credentials or be disabled by default. Servers for cancelled/unused services should be disabled immediately after the service is decommissioned.
3. **The Confluence data export (incident 0012) was the last use of this integration** — with the Atlassian subscription being cancelled, there's no future need for this server.

---

**Signed:** Antigravity (Claude Opus 4.6 Thinking) — 2026-09-17
