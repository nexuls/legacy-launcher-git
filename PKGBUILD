pkgname=legacy-launcher
pkgver=1.40.3
pkgrel=2
pkgdesc="Legacy Launcher for Minecraft"
arch=('any')
url="https://llaun.ch/"
license=('custom')
depends=('java-runtime>=17')
source=(
    "LegacyLauncher.jar::https://llaun.ch/jar"
    'legacy-launcher.svg'
    'legacy-launcher_256.png'
    'legacy-launcher_512.png'
)
# The jar is a zip; without noextract makepkg explodes it into $srcdir.
noextract=('LegacyLauncher.jar')
# Never SKIP the remote source: the endpoint is behind Cloudflare, and an
# unpinned fetch would silently bake a challenge page into the package.
sha256sums=('09ba7517c8a9b30780ff9a6de230b8905247835ae0da914d66991e2b3b4b6c4e'
            'bfe4f259f20bc4626c7619e7d72ec1940e631ff0872258064a1d55e6413f73d2'
            '4d960d32736f6173fc8254b922ebf673e8a2dd2c6dfc963864a3d70b24db8a6b'
            'fd602599d1910358866bcc501118adfd13bcd89bd1a63e23953e68d153160370')

prepare() {
    # Guards the case where a maintainer temporarily sets sha256sums to SKIP
    # while bumping versions: the download endpoint is behind Cloudflare and
    # can answer with an HTML challenge page instead of the jar.
    bsdtar -tf LegacyLauncher.jar >/dev/null 2>&1 \
        || { echo "ERROR: LegacyLauncher.jar is not a valid archive" >&2; return 1; }
}

package() {
    # Install launcher
    install -Dm644 LegacyLauncher.jar \
        "$pkgdir/opt/legacy-launcher/LegacyLauncher.jar"

    # Wrapper script
    install -Dm755 /dev/stdin \
        "$pkgdir/usr/bin/legacy-launcher" <<'EOF'
#!/bin/sh
set -eu
exec java -jar /opt/legacy-launcher/LegacyLauncher.jar "$@"
EOF

    # Desktop entry
    install -Dm644 /dev/stdin \
        "$pkgdir/usr/share/applications/legacy-launcher.desktop" <<'EOF'
[Desktop Entry]
Version=1.0
Type=Application
Name=Legacy Launcher
GenericName=Minecraft Launcher
Comment=Launch and manage Minecraft
Exec=legacy-launcher %U
Icon=legacy-launcher
Terminal=false
Categories=Game;
Keywords=minecraft;
StartupNotify=true
StartupWMClass=org-springframework-boot-loader-PropertiesLauncher
EOF

    # Icons
    install -Dm644 legacy-launcher.svg \
        "$pkgdir/usr/share/icons/hicolor/scalable/apps/legacy-launcher.svg"

    install -Dm644 legacy-launcher_256.png \
        "$pkgdir/usr/share/icons/hicolor/256x256/apps/legacy-launcher.png"

    install -Dm644 legacy-launcher_512.png \
        "$pkgdir/usr/share/icons/hicolor/512x512/apps/legacy-launcher.png"

    # The jar re-execs itself as a second JVM whose main class is Spring Boot's
    # PropertiesLauncher, and AWT derives WM_CLASS from the bottom stack frame,
    # so the window reports org-springframework-boot-loader-PropertiesLauncher.
    # StartupWMClass above covers shells that match on it (GNOME); KDE, XFCE and
    # Cinnamon instead fall back to an icon-theme lookup keyed on WM_CLASS, which
    # these copies satisfy.
    install -Dm644 legacy-launcher_256.png \
        "$pkgdir/usr/share/icons/hicolor/256x256/apps/org-springframework-boot-loader-PropertiesLauncher.png"

    install -Dm644 legacy-launcher_512.png \
        "$pkgdir/usr/share/icons/hicolor/512x512/apps/org-springframework-boot-loader-PropertiesLauncher.png"
}
