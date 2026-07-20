pkgname=legacy-launcher
pkgver=1.40.3
pkgrel=1
pkgdesc="Legacy Launcher for Minecraft"
arch=('any')
url="https://llaun.ch/"
license=('custom')
depends=('java-runtime>=17')
source=(
    "LegacyLauncher.jar::https://llaun.ch/jar"
    'legacy-launcher.svg'
    'legacy-launcher_512.png'
)
# The jar is a zip; without noextract makepkg explodes it into $srcdir.
noextract=('LegacyLauncher.jar')
# Never SKIP the remote source: the endpoint is behind Cloudflare, and an
# unpinned fetch would silently bake a challenge page into the package.
sha256sums=('09ba7517c8a9b30780ff9a6de230b8905247835ae0da914d66991e2b3b4b6c4e'
            'bfe4f259f20bc4626c7619e7d72ec1940e631ff0872258064a1d55e6413f73d2'
            'fd602599d1910358866bcc501118adfd13bcd89bd1a63e23953e68d153160370')

package() {
    # Install launcher
    install -Dm644 LegacyLauncher.jar \
        "$pkgdir/opt/legacy-launcher/LegacyLauncher.jar"

    # Wrapper script
    install -Dm755 /dev/stdin \
        "$pkgdir/usr/bin/legacy-launcher" <<'EOF'
#!/bin/sh
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
StartupNotify=true
EOF

    # Icons
    install -Dm644 legacy-launcher.svg \
        "$pkgdir/usr/share/icons/hicolor/scalable/apps/legacy-launcher.svg"

    install -Dm644 legacy-launcher_512.png \
        "$pkgdir/usr/share/icons/hicolor/512x512/apps/legacy-launcher.png"
}
