# Maintainer: James Bullis <james@ventin.co>
# Repackages the actively maintained claude-desktop-debian release artifacts
# (https://github.com/aaddrick/claude-desktop-debian) for Arch-based systems.
#
# Anthropic switched the official Windows installer to MSIX behind a
# Cloudflare-protected endpoint in February 2026, so extracting it directly
# (as the legacy/ PKGBUILD did) can no longer reach current versions. The
# claude-desktop-debian project maintains that extraction-and-patching
# pipeline and publishes self-contained .deb artifacts (bundled Electron,
# no Debian-specific dependencies), which this PKGBUILD converts into an
# Arch package.
#
# To bump to a new release: check
# https://github.com/aaddrick/claude-desktop-debian/releases for the latest
# tag (vREPACKAGING+claudeAPPVERSION), update pkgver and _relver below to
# match, then run updpkgsums.

pkgname=claude-desktop
pkgver=1.11847.5   # upstream Claude Desktop version (the "claude" part of the release tag)
_relver=2.0.19     # claude-desktop-debian repackaging version (the "v" part of the release tag)
pkgrel=1
pkgdesc="Claude Desktop for Linux – an AI assistant from Anthropic (repackaged from claude-desktop-debian)"
arch=('x86_64' 'aarch64')
url="https://github.com/jmbullis/claude-desktop-arch"
license=('LicenseRef-Anthropic-Commercial-Terms')
depends=('alsa-lib' 'desktop-file-utils' 'gtk3' 'hicolor-icon-theme' 'libnotify'
         'libxss' 'libxtst' 'nss' 'xdg-utils')
optdepends=('bubblewrap: sandbox isolation for Cowork sessions'
            'gnome-keyring: secure credential storage (any org.freedesktop.secrets provider works)'
            'zenity: graphical error dialogs from the launcher')
options=('!strip')  # bundled Electron binaries; stripping breaks them and wastes minutes

_baseurl="https://github.com/aaddrick/claude-desktop-debian/releases/download/v${_relver}%2Bclaude${pkgver}"
_debfile_x86_64="claude-desktop_${pkgver}-${_relver}_amd64.deb"
_debfile_aarch64="claude-desktop_${pkgver}-${_relver}_arm64.deb"

source=("LICENSE")
source_x86_64=("${_debfile_x86_64}::${_baseurl}/${_debfile_x86_64}")
source_aarch64=("${_debfile_aarch64}::${_baseurl}/${_debfile_aarch64}")
noextract=("${_debfile_x86_64}" "${_debfile_aarch64}")

sha256sums=('SKIP')
sha256sums_x86_64=('f4a84a16ecc8d1ab8a8caff058678269bf86b1c7d74bec94b45f7eaf307ab1e5')
sha256sums_aarch64=('4bda08c3833f5405aa2b21e9d76ca5f1dceb859809c1e32a8b85b23c38f11aee')

package() {
  cd "$srcdir"

  local _debfile
  if [[ "$CARCH" == "x86_64" ]]; then
    _debfile="$_debfile_x86_64"
  else
    _debfile="$_debfile_aarch64"
  fi

  # A .deb is an ar archive whose payload lives in data.tar.zst. The payload
  # is already laid out as /usr/... so it extracts straight into pkgdir.
  bsdtar -xOf "$_debfile" data.tar.zst | bsdtar -xf - -C "$pkgdir"

  # Debian's postinst set the SUID bit at install time; on Arch we record it
  # in the package itself. Stock Arch kernels allow unprivileged user
  # namespaces, so Chromium can sandbox without it, but the SUID helper keeps
  # the sandbox working on linux-hardened (kernel.unprivileged_userns_clone=0).
  chmod 4755 "$pkgdir/usr/lib/claude-desktop/node_modules/electron/dist/chrome-sandbox"

  # Unlicense covering this repo's packaging scripts only; the application
  # itself is proprietary Anthropic software subject to their commercial terms.
  install -Dm644 "$srcdir/LICENSE" "$pkgdir/usr/share/licenses/$pkgname/LICENSE.packaging-scripts"
}
