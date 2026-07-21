pkgname=legacy-launcher-git
pkgver=1.40.3
pkgrel=5
pkgdesc="Legacy Launcher for Minecraft (Unofficial Installer)"
arch=('any')
url="https://llaun.ch/"
license=('custom')
depends=('java-runtime>=17')
# Icons live in icons/ and are referenced from $startdir in package() rather
# than listed here: makepkg reduces every local source to its basename
# (get_filename in makepkg's source.sh does ${netfile##*/}), so a path like
# icons/foo.svg is looked up as ./foo.svg and fails. They ship in the same
# repo as this PKGBUILD, so git already covers their integrity.
source=(
    "LegacyLauncher.jar::https://llaun.ch/jar"
)
# The jar is a zip; without noextract makepkg explodes it into $srcdir.
noextract=('LegacyLauncher.jar')
# Never SKIP the remote source: the endpoint is behind Cloudflare, and an
# unpinned fetch would silently bake a challenge page into the package.
sha256sums=('09ba7517c8a9b30780ff9a6de230b8905247835ae0da914d66991e2b3b4b6c4e')

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

    # The game runs in its own JVM and its window is created by GLFW, which sets
    # WM_CLASS to "Minecraft*" independently of anything the launcher does.
    # StartupWMClass holds a single value, so the game needs its own entry.
    # NoDisplay keeps it out of menus and launchers -- it exists only to give
    # shells an app_id -> icon mapping for the game window.
    install -Dm644 /dev/stdin \
        "$pkgdir/usr/share/applications/legacy-launcher-minecraft.desktop" <<'EOF'
[Desktop Entry]
Version=1.0
Type=Application
Name=Minecraft
Comment=Minecraft game window
Exec=legacy-launcher
Icon=legacy-launcher-minecraft
Terminal=false
NoDisplay=true
Categories=Game;
StartupWMClass=Minecraft*
EOF

    # Icons
    install -Dm644 "$startdir/icons/legacy-launcher.svg" \
        "$pkgdir/usr/share/icons/hicolor/scalable/apps/legacy-launcher.svg"

    install -Dm644 "$startdir/icons/legacy-launcher_256.png" \
        "$pkgdir/usr/share/icons/hicolor/256x256/apps/legacy-launcher.png"

    install -Dm644 "$startdir/icons/legacy-launcher_512.png" \
        "$pkgdir/usr/share/icons/hicolor/512x512/apps/legacy-launcher.png"

    # The jar re-execs itself as a second JVM whose main class is Spring Boot's
    # PropertiesLauncher, and AWT derives WM_CLASS from the bottom stack frame,
    # so the window reports org-springframework-boot-loader-PropertiesLauncher.
    # StartupWMClass above covers shells that match on it (GNOME); KDE, XFCE and
    # Cinnamon instead fall back to an icon-theme lookup keyed on WM_CLASS, which
    # these copies satisfy.
    install -Dm644 "$startdir/icons/legacy-launcher_256.png" \
        "$pkgdir/usr/share/icons/hicolor/256x256/apps/org-springframework-boot-loader-PropertiesLauncher.png"

    install -Dm644 "$startdir/icons/legacy-launcher_512.png" \
        "$pkgdir/usr/share/icons/hicolor/512x512/apps/org-springframework-boot-loader-PropertiesLauncher.png"

    # Game icons. Namespaced under legacy-launcher- so they cannot collide with
    # a plain minecraft.png from another package in the shared hicolor theme.
    install -Dm644 "$startdir/icons/minecraft.svg" \
        "$pkgdir/usr/share/icons/hicolor/scalable/apps/legacy-launcher-minecraft.svg"

    install -Dm644 "$startdir/icons/minecraft_256.png" \
        "$pkgdir/usr/share/icons/hicolor/256x256/apps/legacy-launcher-minecraft.png"

    install -Dm644 "$startdir/icons/minecraft_512.png" \
        "$pkgdir/usr/share/icons/hicolor/512x512/apps/legacy-launcher-minecraft.png"

    # Same fallback for the game window's WM_CLASS. The literal asterisk is part
    # of the class GLFW reports, so it is part of the filename too.
    install -Dm644 "$startdir/icons/minecraft_256.png" \
        "$pkgdir/usr/share/icons/hicolor/256x256/apps/Minecraft*.png"

    install -Dm644 "$startdir/icons/minecraft_512.png" \
        "$pkgdir/usr/share/icons/hicolor/512x512/apps/Minecraft*.png"
}
