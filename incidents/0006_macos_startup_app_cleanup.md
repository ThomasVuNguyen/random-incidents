# 0006 — macOS opened too many apps after login

**Date:** 2026-08-31

**Service:** macOS Login Items and launch services  
**Status:** Resolved

![Startup clutter is swept away while three approved utilities remain](0006_macos_startup_app_cleanup.png)

## What happened

After a restart, macOS opened a large collection of applications and background helpers. The visible Login Items contained Notion, Granola, Google Drive, GeminiAppLauncher, Eloquent, Comet, Mailspring, Shottr, and Clicky. Several of those records were stale because their application bundles had already been removed. Additional launch services started Epic Games Launcher, GOG Galaxy, Docker, Comet updater, Glaze CrossAppBroker, Atlas updater, Polar updater, and Steam cleanup. macOS session restoration could also reopen applications that had been running before restart.

The desired startup set was only Google Drive, Tailscale, and Shottr. Clicky and Mailspring were to be uninstalled, and ChatGPT Atlas was to be deleted.

## What I did to fix it

I removed every visible Login Item except Google Drive and Shottr, while verifying that Tailscale's separate login helper remained enabled. This removed the active entries for Notion and Comet and cleared stale entries for Granola, GeminiAppLauncher, Eloquent, Clicky, and Mailspring.

I disabled the user-level auto-launch helpers for Comet, Glaze, Docker, Epic Games Launcher, GOG Galaxy, ChatGPT Atlas, Polar, and Steam. After macOS administrator authentication, I also disabled the Docker socket and network daemons and the GOG client service at the system level. Google updater services were preserved because Google Drive remains an approved startup application.

Mailspring was stopped and its application bundle was moved to a dedicated folder in Trash. Clicky's application bundle was already absent, so its stale Login Item was removed. ChatGPT Atlas's app bundle was also already absent; I disabled and removed its updater plus its remaining application support, browser profile, preferences, and cache. The removed files remain recoverable at:

```text
/Users/thomasthemaker/.Trash/codex-startup-cleanup-2026-08-31-01
```

Finally, I disabled both global and host-specific macOS application relaunch settings so previously open apps do not return automatically at the next login.

Verification confirmed:

- Visible Login Items: Google Drive and Shottr only.
- Tailscale login helper: enabled.
- Mailspring and ChatGPT Atlas processes: not running.
- Clicky, Mailspring, and ChatGPT Atlas app bundles: absent from Applications.
- Docker and GOG user and system startup helpers: disabled.
- macOS session restoration at login: disabled.

## What should prevent it next time

- Review **System Settings → General → Login Items & Extensions** after installing productivity apps, browsers, game launchers, or developer tools.
- Keep auto-launch enabled only for utilities that must be available immediately after login.
- Disable an application's own “launch at login” preference during installation when it is not required.
- Uninstall apps through their built-in uninstall flow when available so stale Login Items and update helpers are removed with the app.
- Leave macOS application relaunch disabled if a clean desktop after restart is preferred.

— Codex, 2026-08-31
