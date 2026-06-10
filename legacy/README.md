# Legacy PKGBUILD (frozen at Claude Desktop 0.14.10)

This is the original approach from the archived upstream repo: download the
official Windows Squirrel `.exe` installer, extract it with 7zip, patch the
app for Linux, and bundle Electron via npm.

It still builds — but in February 2026 Anthropic switched the Windows
installer to MSIX behind a Cloudflare-protected endpoint, so the legacy
download URL used here is permanently frozen at the last Squirrel build,
**Claude Desktop 0.14.10 (October 2025)**. An eight-month-old (and counting)
client may stop being accepted by Anthropic's servers at any time.

Use the PKGBUILD at the repository root instead, which repackages the
actively maintained
[claude-desktop-debian](https://github.com/aaddrick/claude-desktop-debian)
artifacts and tracks current 1.x releases.

To build this legacy version anyway:

```bash
cd legacy
makepkg -si
```

Relative to the archived upstream, this copy has three fixes so 0.14.10
actually works: Electron pinned to the 37.x line the app targets, an
`AuthRequest` stub added to the `claude-native` shim (its absence broke
browser login), and the stale `main_window.tgz` title-bar overwrite removed.
