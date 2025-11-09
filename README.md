# flatwrap

![flatwrap logo](flatwrap_logo.png)

**Automatic executable wrappers for Flatpak applications**

flatwrap creates convenient command-line shortcuts for your Flatpak apps, making them as easy to launch as native binaries. No more typing `flatpak run com.example.App` — just type `app` instead.

## Features

- 🚀 **Zero-friction access** — Launch Flatpak apps directly from your terminal
- 🔄 **Auto-sync** — Automatically updates wrappers when you install or remove Flatpak apps
- 🎯 **Smart aliasing** — Creates both full app IDs and short aliases (e.g., `firefox` for `org.mozilla.firefox`)
- ⚡ **Collision-safe** — Handles duplicate names with stable hash suffixes
- 🔧 **Set and forget** — Uses systemd to keep wrappers in sync automatically

## Quick Start

```bash
# Install flatwrap itself
curl -o ~/.local/bin/flatwrap https://raw.githubusercontent.com/fadenb/flatwrap/main/flatwrap
chmod +x ~/.local/bin/flatwrap

# Set up automatic syncing
flatwrap install

# That's it! Your Flatpak apps are now available as commands
```

After installation, you might need to start a new terminal session for the PATH changes to take effect.

## Usage

### Commands

```bash
flatwrap install    # Set up PATH, enable auto-sync, and generate all wrappers
flatwrap sync       # Manually regenerate wrappers from current Flatpak apps
flatwrap list       # Show all available wrapper commands
flatwrap uninstall  # Disable auto-sync (wrappers remain in place)
flatwrap help       # Display help information
```

### Examples

```bash
# After installation, launch apps naturally:
firefox
spotify
gimp

# All arguments are passed through:
code ~/my-project
vlc movie.mp4 --fullscreen

# Full app IDs also work:
org.mozilla.firefox
com.spotify.Client
```

## How It Works

1. **Wrapper Generation** — Creates executable scripts in `~/.local/bin/` that call `flatpak run <app-id>`
2. **Intelligent Naming** — Generates both full app IDs (`org.mozilla.firefox-flatwrap`) and short aliases (`firefox`)
3. **Collision Handling** — When multiple apps have the same short name, adds a stable hash suffix
4. **Automatic Sync** — Monitors Flatpak directories with systemd path units and runs an hourly safety timer
5. **PATH Integration** — Ensures `~/.local/bin/` is in your PATH

### File Locations

- **Wrappers**: `~/.local/bin/*-flatwrap`
- **Systemd units**: `~/.config/systemd/user/flatwrap.{service,path,timer}`
- **Auto-sync triggers**:
  - `~/.local/share/flatpak/exports/share/applications`
  - `/var/lib/flatpak/exports/share/applications`

## Requirements

- bash
- flatpak
- systemd (for automatic syncing)
- One of: `sha256sum`, `shasum`, or `openssl` (for collision handling)

## Technical Details

### Naming Convention

Wrappers use a `-flatwrap` suffix to avoid conflicts with system binaries:

- Full ID wrapper: `org.mozilla.firefox-flatwrap`
- Short alias symlink: `firefox` → `firefox-flatwrap`
- Collision resolution: `firefox-a1b2c3-flatwrap`

### Auto-Sync Mechanism

flatwrap uses two systemd triggers:

1. **Path unit** — Activates when Flatpak export directories change
2. **Timer unit** — Runs hourly as a safety net

Both trigger the `flatwrap.service` which runs `flatwrap sync` to regenerate wrappers.

### Dry Run Mode

Test changes without modifying your system:

```bash
flatwrap sync --dry-run
```

## Troubleshooting

**Wrappers not in PATH**

Start a new terminal session or run:
```bash
source ~/.profile
```

**Auto-sync not working**

Check systemd status:
```bash
systemctl --user status flatwrap.path
systemctl --user status flatwrap.timer
```

**Command conflicts**

If a wrapper conflicts with an existing command, flatwrap will create a hashed version. Use `flatwrap list` to see the actual command names.

## Uninstallation

```bash
# Disable auto-sync
flatwrap uninstall

# Remove wrappers (optional)
rm -f ~/.local/bin/*-flatwrap

# Remove flatwrap itself (optional)
rm -f ~/.local/bin/flatwrap
```

---

**Made with ❤️ for the Flatpak community**
