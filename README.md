# ⚠️ Fork status (June 2026)

This is a fork of the archived [aaddrick/claude-desktop-arch](https://github.com/aaddrick/claude-desktop-arch) repo, updated so the build still works — with one important caveat:

**This PKGBUILD installs Claude Desktop 0.14.10 (October 2025) and can never install anything newer.** In February 2026 Anthropic switched the Windows installer from the Squirrel `.exe` format (which this build extracts) to MSIX, served from a Cloudflare-protected endpoint. The legacy download URL used here still works, but it is frozen at the last Squirrel build, 0.14.10. Supporting current 1.x versions would mean porting MSIX extraction, headless-browser download-URL resolution, and a newer patch set — essentially what [aaddrick/claude-desktop-debian](https://github.com/aaddrick/claude-desktop-debian) now does.

**If you want a current, maintained Claude Desktop on Arch/EndeavourOS, use one of these instead:**

* [`claude-desktop-native`](https://aur.archlinux.org/packages/claude-desktop-native) on the AUR ([upstream repo](https://github.com/jkoelker/claude-desktop-native)) — `yay -S claude-desktop-native`
* [aaddrick/claude-desktop-debian](https://github.com/aaddrick/claude-desktop-debian) — actively maintained; its AppImage builds run fine on Arch-based distros

Changes made in this fork relative to the archived upstream:

* Pinned Electron to the 37.x line (Claude Desktop 0.14.10 targets Electron 37.6.0; unpinned "latest" has drifted several majors ahead).
* Added an `AuthRequest` stub to the `claude-native` shim. 0.14.10 calls `AuthRequest.isAvailable()` during login; without the stub that check throws and browser-based login can fail.
* Removed the title-bar fix downloaded from `emsi/claude-desktop` (`main_window.tgz`). It replaced the app's title-bar renderer with a snapshot built against an older app version, which no longer matches 0.14.10's main process. 0.14.10 ships its own title-bar renderer.
* `LICENSE` is sourced from this repo instead of being downloaded from the archived upstream.
* Switched from the deprecated `asar` npm package to `@electron/asar`.

Also note: a 0.14.10 client is eight months old. It may still sign in and chat, but expect missing features and the possibility that Anthropic eventually blocks old clients.

---

**Debian/Ubuntu Linux users:** For the DPKG/AppImage build script and Debian-specific instructions: [https://github.com/aaddrick/claude-desktop-debian](https://github.com/aaddrick/claude-desktop-debian)

***THIS IS AN UNOFFICIAL PKGBUILD FOR ARCH LINUX BASED SYSTEMS!***

If you run into an issue with this build script, make an issue here. Don't bug Anthropic about it - they already have enough on their plates.

# Claude Desktop for Linux (Arch PKGBUILD)

This project provides an Arch Linux `PKGBUILD` to install the Claude Desktop application. It was inspired by [k3d3's claude-desktop-linux-flake](https://github.com/k3d3/claude-desktop-linux-flake) and their [Reddit post](https://www.reddit.com/r/ClaudeAI/comments/1hgsmpq/i_successfully_ran_claude_desktop_natively_on/) about running Claude Desktop natively on Linux.

Supports MCP! Location of the MCP-configuration file is: `~/.config/Claude/claude_desktop_config.json`

![image](https://github.com/user-attachments/assets/93080028-6f71-48bd-8e59-5149d148cd45)

Supports the Ctrl+Alt+Space popup!
![image](https://github.com/user-attachments/assets/1deb4604-4c06-4e4b-b63f-7f6ef9ef28c1)

Supports the Tray menu! (Screenshot of running on KDE)
![image](https://github.com/user-attachments/assets/ba209824-8afb-437c-a944-b53fd9ecd559)

# Building & Installation (Arch Linux)

For Arch Linux and Arch-based distributions, you can build and install using the provided `PKGBUILD`:

```bash
# Clone this repository
git clone https://github.com/jmbullis/claude-desktop-arch.git
cd claude-desktop-arch

# Update checksums (needed once, or after PKGBUILD/install script changes)
updpkgsums

# Build and install the package
# This command automatically handles dependencies, builds, and installs
# Use makepkg -sci to automatically clean up build files afterwards
makepkg -si
```

The `PKGBUILD` will automatically:
 - Detect your architecture (x86_64 or aarch64)
 - Determine the latest application version
 - Download the correct official Windows installer
 - Check for and prompt to install required dependencies (using `pacman`)
 - Extract application resources
 - Apply Linux compatibility patches (native module stub, Wayland flags)
 - Package Electron locally within the application
 - Create and install an Arch Linux package (`.pkg.tar.zst`)

# Uninstallation

If you installed the package using `makepkg -si` or `pacman -U`, you can uninstall it using `pacman`:

```bash
sudo pacman -R claude-desktop
```

To also remove user-specific configuration files (including MCP settings):

```bash
sudo pacman -Rns claude-desktop
# Or manually remove the config directory:
# rm -rf ~/.config/Claude
```

# Troubleshooting

## Window Scaling Issues

If your window isn't scaling correctly the first time or two you open the application, right-click on the claude-desktop panel (taskbar) icon and choose "Quit". When doing a safe shutdown like this, the application saves window state information to `~/.config/Claude/` which should resolve the issue on subsequent launches. Force quitting the application (e.g., via `kill`) will not trigger these state updates.

## Application Fails to Launch (Sandbox Issues)

The `PKGBUILD` attempts to configure the Electron sandbox correctly by packaging Electron locally and setting SUID permissions on `chrome-sandbox` via the `.install` script. This should work in most cases.

However, if the application installs but fails to launch (you might see errors related to sandboxing or zygote processes in the terminal when running `claude-desktop`), you *could* try launching it with the `--no-sandbox` flag as a last resort:

```bash
claude-desktop --no-sandbox
```

If this works, you *could* make it permanent by editing the launcher script `/usr/bin/claude-desktop`, but **this is strongly discouraged** as it reduces security isolation. It's better to investigate *why* the sandbox isn't working (e.g., kernel settings like `kernel.unprivileged_userns_clone`, filesystem mount options like `nosuid` on `/usr/lib`).

**Editing the launcher (Use with caution):**
1.  Open the launcher script with root privileges: `sudo nano /usr/bin/claude-desktop`
2.  Locate the line near the end that executes Electron. It will look like this:
    ```bash
    "$ELECTRON_EXEC" "${ELECTRON_ARGS[@]}"
    ```
3.  Modify that line by adding `--no-sandbox` immediately after `"$ELECTRON_EXEC"`:
    ```bash
    # Before:
    # "$ELECTRON_EXEC" "${ELECTRON_ARGS[@]}"
    # After:
    "$ELECTRON_EXEC" --no-sandbox "${ELECTRON_ARGS[@]}"
    ```
4.  Save the file (Ctrl+O in nano, then Enter) and exit (Ctrl+X in nano).

# How it works (Arch PKGBUILD)

Claude Desktop is an Electron application packaged as a Windows executable. The `PKGBUILD` performs several key operations to make it work natively on Arch Linux:

1.  **Source Fetching:** Downloads the official Windows installer (`.exe`) appropriate for the target architecture (`$CARCH`).
2.  **Version Detection:** The `pkgver()` function extracts the version number from the downloaded installer's contents (specifically, the embedded `.nupkg` filename).
3.  **Build Process (`build()`):**
    *   Installs `electron` and `asar` locally using `npm` to ensure correct versions for packaging.
    *   Extracts the Windows installer (`.exe`) using `p7zip`.
    *   Extracts the application files from the embedded NuGet package (`.nupkg`).
    *   Extracts icons using `wrestool` and `icotool`.
    *   Unpacks the `app.asar` archive using the local `asar` tool.
    *   Replaces the Windows-specific native module (`claude-native`) with a Linux-compatible JavaScript stub (in both `app.asar.contents` and `app.asar.unpacked`). This stub provides dummy functions for Windows APIs while keeping necessary parts like `KeyboardKey`, plus an `AuthRequest` class whose `isAvailable()` returns `false` so login falls back to the system browser.
    *   Repacks the modified `app.asar`.
4.  **Packaging (`package()`):**
    *   Installs the application files (`app.asar`, `app.asar.unpacked`) into `/usr/lib/claude-desktop/`.
    *   Installs the locally built `node_modules` (containing Electron) into `/usr/lib/claude-desktop/`.
    *   Installs icons into `/usr/share/icons/hicolor/`.
    *   Creates a `.desktop` file in `/usr/share/applications/` for menu integration.
    *   Creates a launcher script `/usr/bin/claude-desktop` that:
        *   Uses the packaged Electron executable.
        *   Detects Wayland sessions and adds necessary flags (`--enable-features=UseOzonePlatform,WaylandWindowDecorations --ozone-platform=wayland`).
        *   Changes to the application directory before launching.
5.  **Post-Install (`claude-desktop.install`):**
    *   Sets SUID permissions (`chown root:root`, `chmod 4755`) on the packaged `chrome-sandbox` executable (`/usr/lib/claude-desktop/node_modules/electron/dist/chrome-sandbox`) to allow the sandbox to function correctly.
    *   Updates the desktop file database (`update-desktop-database`).

This process works because the core Claude Desktop application is largely cross-platform (being Electron-based). The main challenge is handling the Windows-specific native module and ensuring the Electron environment (including the sandbox) is set up correctly on Linux.

# References & Alternatives

*   **NixOS:** For NixOS users, please refer to [k3d3's claude-desktop-linux-flake](https://github.com/k3d3/claude-desktop-linux-flake) repository.
*   **Debian/Ubuntu/AppImage:** For a build script targeting `.deb` or `.AppImage` formats, see [aaddrick/claude-desktop-debian](https://github.com/aaddrick/claude-desktop-debian).
*   **Emsi's Fork:** [emsi/claude-desktop](https://github.com/emsi/claude-desktop) contains related work and the title bar fix used here.

# License

The `PKGBUILD` and `.install` script in this repository are provided under the Unlicense. See the [LICENSE](LICENSE) file.

The Claude Desktop application itself, downloaded by the `PKGBUILD`, is proprietary software developed by Anthropic and is subject to their terms of service. This project only provides a way to package and run it on Arch Linux.
