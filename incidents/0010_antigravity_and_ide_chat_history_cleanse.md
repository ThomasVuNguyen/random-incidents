# 0010 — Antigravity & Antigravity IDE Chat History Cleanse and Archival

![Antigravity Chat History Cleanse](0010_antigravity_and_ide_chat_history_cleanse.png)

## What happened

Over time, Antigravity Desktop and Antigravity IDE accumulated massive volumes of conversational state, protobuf conversation binaries, workspace brain trajectories, and cached workspace data:

- **Antigravity Desktop**: Accumulated 369 conversation protobuf files, 355 brain workspaces, 227 annotation records, and an inflated `agyhub_summaries_proto.pb` and `app_storage.json` (~1.68 GB total).
- **Antigravity IDE**: Accumulated 1,418 conversation protobuf files, 481 brain workspaces, 102 implicit context items, and 149 active `workspaceStorage` records (~3.06 GB total).
- **Performance impact**: Across both environments, 1,787 chat session files and over 830 brain workspaces were consuming significant disk space, degrading session lookup performance, and cluttering the sidebar interface with obsolete sessions.

## What was done to fix it

### 1. Pre-Cleanse Safety and Graceful Shutdown
- Gracefully shut down active `Antigravity.app` processes via AppleScript to cleanly release file locks and prevent in-memory cache states from writing back to disk upon process exit.
- Verified process boundaries to isolate the active CLI session (`~/.gemini/antigravity-cli`), guaranteeing uninterrupted agent execution.

### 2. Full Archival to `~/.gemini/backups/` (`20260906_030500`)
Before removing any session data, complete compressed archives and backups were created and verified:

| Archive / Backup File | Size | Contents |
|-----------------------|------|----------|
| `antigravity-desktop-backup-20260906_030500.tar.gz` | 607.15 MB | Desktop `conversations` (369), `brain` (355), `annotations`, `antigravity_state.pbtxt`, `agyhub_summaries_proto.pb` |
| `antigravity-ide-backup-20260906_030500.tar.gz` | 1,608.61 MB | IDE `conversations` (1,418), `brain` (481), `implicit` context data |
| `ide-workspaceStorage-backup-20260906_030500.tar.gz` | 3.00 MB | IDE `User/workspaceStorage` (149 workspaces) and file edit `User/History` |
| `projects-backup-20260906_030500.tar.gz` | 0.01 MB | All registered project configurations and `projects.json` |
| `desktop-app_storage.json-20260906_030500.bak` | 130 KB | Desktop application sidebar and session layout state |
| `agyhub_summaries_proto-20260906_030500.pb.bak` | 365 KB | Antigravity Desktop chat summaries protobuf database |

### 3. Execution of the Cleanse
Once the integrity of all backup archives was verified, the reset was performed:
- **Antigravity Desktop**: Cleared all 369 `.pb` conversations, 355 brain workspaces, 227 annotations, crash logs, and scratch files. Removed obsolete protobuf chat summary databases. Cleared Chromium/Electron GPU and code caches. Reset `app_storage.json` to a pristine initial configuration.
- **Antigravity IDE**: Cleared all 1,418 `.pb` conversations, 481 brain workspaces, 102 implicit context items, crash logs, scratch files, and 149 `workspaceStorage` and `History` caches.
- **Projects**: Reset `~/.gemini/projects.json` to `{"projects": {}}` and emptied obsolete project definitions.
- **Preserved Core Assets**: System configurations (`settings.json`), MCP definitions (`mcp_config.json`), skills, global workflows, OAuth credentials, and installed extension packages remained fully intact.

All target directories were verified post-cleanse with 0 active chat/session items.

## What will prevent this from happening again

1. **Automated Archival & Rotation Script**: A reusable backup and cleanse routine can be scheduled periodically (e.g. quarterly or via a maintenance slash command) to prevent uncompressed conversation buildup.
2. **Offsite Archival Sync**: Include `~/.gemini/backups/` in scheduled cloud storage syncs (e.g., the GCS backup bucket configured in Incident 0009) to preserve historical logs without local bloat.
3. **Session Retention Policy**: Antigravity workspaces benefit from periodic pruning of scratch files and finished brain trajectories to maintain peak IDE responsiveness.

---

**Signed:** antigravity gemini 3.8 flash · 2026-09-06
