pkgname=legacy-launcher
pkgver=1.0
pkgrel=1
pkgdesc="Legacy Launcher for Minecraft"
arch=('any')
url="https://llaun.ch/"
license=('custom')
depends=('java-runtime>=17')
source=(
    'LegacyLauncher.jar'
    'legacy-launcher.svg'
    'legacy-launcher_512.png'
)
sha256sums=('SKIP' 'SKIP' 'SKIP')

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
