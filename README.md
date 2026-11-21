# mac-workspace — chezmoi source

Software binaries are installed separately on each Mac. This repo restores **configuration and plugin declarations** only.

## Bootstrap a new Mac

```sh
# 1. Install the programs themselves (Homebrew is assumed)
brew install chezmoi
brew install --cask herdr                 # Herdr desktop app
curl -fsSL https://pi.dev/install.sh | sh # pi
curl -fsSL https://opencode.ai/install | sh # opencode  (or your usual installer)

# 2. Pull and apply all config; the after-script installs plugins/extensions
chezmoi init --apply git@github.com:lcinshu/mac-workspace.git

# 3. Log in per machine (credentials are intentionally not synced)
pi        -> /login     (opencode-go)
opencode  -> /connect / auth
herdr     -> no login needed
```

`chezmoi init --apply` clones into `~/.local/share/chezmoi` and runs the state.
The `modify_` templates merge the managed keys into files that the programs also write, so pi/Herdr can keep updating their own fields.

## Daily use

```sh
chezmoi diff      # review before applying
chezmoi apply     # apply local changes
chezmoi update    # git pull + apply (on the other Macs)
chezmoi re-add ~/.config/herdr/config.toml   # after editing a plain config file
```

Plugin declarations live in two places:
- pi: `dot_pi/agent/modify_settings.json.tmpl` (`packages` array). On apply, `pi update --extensions` reconciles the installs.
- herdr: `dot_config/herdr/herdr-plugins.txt` (`owner/repo|ref|plugin-id|state`). The `run_onchange_after` script reinstalls pinned refs and enables/disables each plugin.
- opencode: `package.json` + `package-lock.json` in `dot_config/opencode`, installed with `npm install`.

The after-script is `.chezmoiscripts/run_onchange_after_10-plugins.sh.tmpl`; it re-runs whenever the plugin list or its own content changes.

## What is intentionally NOT synced

- Credentials: `~/.pi/agent/auth.json`, opencode `service.json` (contains a password).
- Runtime state: pi sessions, Herdr core sockets / `terminals.json`, Herdr App `view.json` / `shared.json` / `installed.json`, logs, lock files.
- Installed artifacts: `node_modules/`, pi `install/` and `npm/`, Herdr plugin git checkouts under `plugins/github/`.
- Machine-local extension: `~/.local/share/opencode-console/cli/pi-extension.js` is referenced in `extensions`; it survives on this Mac via the modify-template and is deliberately not recreated elsewhere. Install opencode-console on any Mac that needs it.

Live workspaces / running agents are shared by herdr's own remote-machine feature (`hc machine add user@host --name box`), not by file sync.

## Inventory captured (2026-11-21)

- pi: 17 packages, provider `opencode-go`, model `hy4`, theme `tokyo-night`.
- herdr TUI: `config.toml`, 14 pinned plugins (1 disabled), 8 human-edited plugin config files.
- herdr App: `settings.json`, `keys.json`, 3 layouts, 2 themes. Shipped mods are left to the app.
- opencode: `config.json`, `opencode.json`, `cli.json`, `package.json`, `package-lock.json`, 2 disabled plugin sources.
