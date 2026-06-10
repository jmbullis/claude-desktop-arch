# Claude Desktop for Arch Linux (PKGBUILD)

***THIS IS AN UNOFFICIAL PKGBUILD FOR ARCH LINUX BASED SYSTEMS (Arch, EndeavourOS, CachyOS, Manjaro, ...)***

This fork of the archived [aaddrick/claude-desktop-arch](https://github.com/aaddrick/claude-desktop-arch) installs the **current** Claude Desktop (1.x) by repackaging the release artifacts of the actively maintained [aaddrick/claude-desktop-debian](https://github.com/aaddrick/claude-desktop-debian) project into a native Arch package.

If you run into an issue with this build script, make an issue here. Don't bug Anthropic about it - they already have enough on their plates.

## Why repackage the Debian build?

The original PKGBUILD downloaded the official Windows installer (Squirrel `.exe`), extracted it with 7zip, and patched the app for Linux. In February 2026 Anthropic switched the Windows installer to MSIX, served from a Cloudflare-protected endpoint, so that pipeline can no longer reach current versions — the legacy URL is frozen at Claude Desktop 0.14.10 (October 2025).

The claude-desktop-debian project maintains the full extraction-and-patching pipeline (MSIX extraction, headless-browser download resolution, Linux compatibility patches, bundled Electron) and publishes self-contained `.deb` artifacts with **no Debian-specific dependencies** — everything needed is bundled. This PKGBUILD simply converts those artifacts into a proper Arch package. The old self-contained PKGBUILD is preserved in [`legacy/`](legacy/) if you want the historical approach (it still builds, but can only ever install 0.14.10).

Supports MCP! Location of the MCP-configuration file is: `~/.config/Claude/claude_desktop_config.json`

## Building & Installation

```bash
# Clone this repository
git clone https://github.com/jmbullis/claude-desktop-arch.git
cd claude-desktop-arch

# Build and install the package
makepkg -si
```

The PKGBUILD will:
 - Detect your architecture (x86_64 or aarch64)
 - Download the matching `.deb` release artifact from claude-desktop-debian (checksums pinned)
 - Extract it into a native Arch package (bundled Electron included)
 - Record the SUID bit on `chrome-sandbox` so the Chromium sandbox also works on `linux-hardened`

To update to a new Claude Desktop release: check the [claude-desktop-debian releases page](https://github.com/aaddrick/claude-desktop-debian/releases) for the latest tag (`vREPACKAGING+claudeAPPVERSION`), set `pkgver` and `_relver` in the PKGBUILD to match, run `updpkgsums`, then `makepkg -si` again.

Optional extras:
 - `bubblewrap` — sandbox isolation for Cowork sessions (`sudo pacman -S bubblewrap`)
 - `gnome-keyring` (or any `org.freedesktop.secrets` provider) — secure credential storage

## Uninstallation

```bash
sudo pacman -R claude-desktop
# To also remove user configuration (including MCP settings):
# rm -rf ~/.config/Claude
```

## Troubleshooting

The launcher has a built-in diagnostic mode:

```bash
claude-desktop --doctor
```

Launcher logs are written to `~/.cache/claude-desktop-debian/` (the directory name comes from the upstream packaging project). The launcher auto-detects Wayland and applies the appropriate Electron/Ozone flags, and automatically falls back to `--disable-gpu` after a GPU crash (set `CLAUDE_DISABLE_GPU=0` to retest hardware acceleration).

For anything app-related (rendering, tray, title bar, Cowork), check the upstream project's [documentation and issues](https://github.com/aaddrick/claude-desktop-debian) — this repo only converts their artifact into an Arch package.

## How it works

1. **Source:** Downloads the architecture-appropriate `.deb` from claude-desktop-debian's GitHub releases, with pinned sha256 checksums.
2. **Packaging:** A `.deb` is an `ar` archive whose payload is `data.tar.zst`, already laid out as `/usr/...`. The PKGBUILD streams it straight into `$pkgdir` with `bsdtar`:
   - `/usr/bin/claude-desktop` — launcher script (Wayland detection, GPU-crash recovery, `--doctor`)
   - `/usr/lib/claude-desktop/` — bundled Electron with `app.asar` in its `resources/` directory
   - `/usr/share/applications/`, `/usr/share/icons/hicolor/`, `/usr/share/metainfo/` — desktop integration
3. **Permissions:** `chrome-sandbox` gets mode `4755` recorded in the package. On stock Arch kernels Chromium uses unprivileged user namespaces and doesn't need it; on `linux-hardened` (where `kernel.unprivileged_userns_clone=0`) the SUID helper keeps the sandbox functional.
4. **Desktop integration:** pacman's standard hooks (`update-desktop-database`, icon cache) handle registration of the `.desktop` file and the `claude://` URL scheme — no `.install` script needed.

Note: the Debian package's postinst also installs AppArmor profiles to work around Ubuntu 24.04+'s `apparmor_restrict_unprivileged_userns` restriction. Arch doesn't enable that restriction, so the profiles aren't needed here. If you run a custom AppArmor setup that restricts user namespaces, see the upstream project's notes.

## References & Alternatives

*   **claude-desktop-debian:** [aaddrick/claude-desktop-debian](https://github.com/aaddrick/claude-desktop-debian) — the upstream packaging project this PKGBUILD repackages. Also publishes AppImages that run on Arch directly.
*   **AUR:** [`claude-desktop-native`](https://aur.archlinux.org/packages/claude-desktop-native) ([upstream](https://github.com/jkoelker/claude-desktop-native)) and [`claude-desktop-bin`](https://aur.archlinux.org/packages/claude-desktop-bin) are maintained AUR alternatives.
*   **NixOS:** [k3d3's claude-desktop-linux-flake](https://github.com/k3d3/claude-desktop-linux-flake), and claude-desktop-debian ships its own Nix flake.
*   **Legacy:** [`legacy/`](legacy/) contains the original archived PKGBUILD (Windows-installer extraction, frozen at Claude Desktop 0.14.10).

## License

The `PKGBUILD` and scripts in this repository are provided under the Unlicense. See the [LICENSE](LICENSE) file.

The Claude Desktop application itself is proprietary software developed by Anthropic and subject to their terms of service. This project only provides a way to package and run it on Arch Linux.
