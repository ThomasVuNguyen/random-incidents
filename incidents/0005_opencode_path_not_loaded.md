# 0005 — OpenCode installed but the active shell could not find it

**Date:** 2026-08-27

**Service:** OpenCode CLI on macOS zsh  
**Status:** Resolved

![A terminal command becomes available after the shell reloads its PATH](0005_opencode_path_not_loaded.png)

## What happened

OpenCode `1.18.23` installed successfully, but running `opencode` immediately afterward in the same Terminal session returned `zsh: command not found: opencode`.

The installer had placed the executable at `/Users/thomasthemaker/.opencode/bin/opencode` and appended that directory to `/Users/thomasthemaker/.zshrc`. The active zsh process had started before that change, so its in-memory `PATH` did not yet include the new directory.

## What I did to fix it

I inspected the executable and shell configuration, confirmed that the installer added the correct PATH entry, and verified the executable was present and executable. I then started a fresh interactive zsh—the equivalent of reloading `.zshrc` or opening a new Terminal window—and ran a smoke test:

```text
/Users/thomasthemaker/.opencode/bin/opencode
1.18.23
```

No reinstall or `.zshrc` edit was necessary. The existing Terminal tab can load the already-correct configuration with `source ~/.zshrc`; future Terminal windows will load it automatically.

## What should prevent it next time

- After a CLI installer updates a shell startup file, open a new Terminal window or reload that file before testing the command.
- Have installers print a final copy-paste command such as `source ~/.zshrc` when they change PATH for the current shell.
- Verify both the executable location and a fresh interactive shell before reinstalling a CLI that reports `command not found` immediately after installation.

— Codex, 2026-08-27
