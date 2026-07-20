<div align="center">

<img src="legacy-launcher.svg" alt="Legacy Launcher" width="96" height="96">

# Legacy Launcher PKGBUILD

**An unofficial Arch Linux package for installing [Legacy Launcher](https://llaun.ch/) as a native application.**

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Arch Linux](https://img.shields.io/badge/Arch_Linux-1793D1?logo=arch-linux&logoColor=white)](https://archlinux.org/)
[![Java 17+](https://img.shields.io/badge/Java-17%2B-orange.svg)](https://openjdk.org/)

</div>

---

This package installs the official Legacy Launcher JAR under `/opt`, creates a launcher script, installs a desktop entry, and registers application icons so it integrates cleanly with your desktop environment.

> [!NOTE]
> This repository is **not affiliated with the Legacy Launcher project**. It simply packages the official launcher for Arch Linux. All rights to the launcher itself belong to its original authors.

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [What Gets Installed](#what-gets-installed)
- [Usage](#usage)
- [Updating Legacy Launcher](#updating-legacy-launcher)
- [Uninstall](#uninstall)
- [Repository Structure](#repository-structure)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Features

- ✅ Native Arch package built with `makepkg`
- 🖥️ Desktop menu integration via a `.desktop` entry
- 🎨 Scalable SVG and high-resolution PNG application icons
- ⌨️ Launcher available as the `legacy-launcher` command
- 📁 Installs to standard Linux locations (`/opt`, `/usr/bin`, `/usr/share`)
- 🧹 Clean removal through `pacman`

## Requirements

| Requirement | Notes |
| --- | --- |
| Arch Linux (or an Arch-based distro) | e.g. Manjaro, EndeavourOS |
| `base-devel` | Provides `makepkg` and build tooling |
| Java 17 or newer | Runtime dependency (`java-runtime>=17`) |

Install the required packages:

```bash
sudo pacman -S --needed base-devel jre17-openjdk
```

## Installation

Clone the repository and build the package:

```bash
git clone <repository-url>
cd legacy-launcher
makepkg -si
```

`makepkg -si` builds the package and installs it (along with any missing dependencies) via `pacman`. When it finishes, Legacy Launcher is available in your application menu and on the command line.

> [!TIP]
> To build the package **without** installing it, run `makepkg` on its own. The resulting `legacy-launcher-*.pkg.tar.zst` can then be installed later with `sudo pacman -U <file>`.

## What Gets Installed

| Path | Description |
| --- | --- |
| `/opt/legacy-launcher/LegacyLauncher.jar` | The launcher application |
| `/usr/bin/legacy-launcher` | Wrapper script (`java -jar …`) |
| `/usr/share/applications/legacy-launcher.desktop` | Desktop menu entry |
| `/usr/share/icons/hicolor/scalable/apps/legacy-launcher.svg` | Scalable icon |
| `/usr/share/icons/hicolor/512x512/apps/legacy-launcher.png` | Raster icon |

## Usage

Once installed, launch Legacy Launcher from:

- **Your application launcher** — search for *Legacy Launcher*
- **The command line:**

  ```bash
  legacy-launcher
  ```

Any arguments you pass are forwarded to the underlying JAR:

```bash
legacy-launcher --help
```

## Updating Legacy Launcher

When a new version of Legacy Launcher is released:

1. Download the latest **LegacyLauncher.jar** from the [official website](https://llaun.ch/).
2. Replace the existing `LegacyLauncher.jar` in this repository with the new one.
3. Bump `pkgver` in the `PKGBUILD` if you want the package version to reflect the update.
4. Rebuild and reinstall:

   ```bash
   makepkg -si
   ```

The new launcher overwrites the previous installation.

## Uninstall

Remove the package with `pacman`:

```bash
sudo pacman -R legacy-launcher
```

This removes all files installed by the package. Your personal Legacy Launcher data and game files (stored in your home directory) are left untouched.

## Repository Structure

```
.
├── PKGBUILD                  # Build recipe for makepkg
├── LegacyLauncher.jar        # Official launcher (upstream)
├── legacy-launcher.svg       # Scalable application icon
├── legacy-launcher_256.png   # 256×256 raster icon
├── legacy-launcher_512.png   # 512×512 raster icon
├── LICENSE                   # MIT license for packaging files
└── README.md                 # This file
```

## Troubleshooting

<details>
<summary><strong>The <code>legacy-launcher</code> command isn't found after install</strong></summary>

Make sure `/usr/bin` is in your `PATH` (it is by default). Try opening a new shell, or run `hash -r` to refresh your shell's command cache.
</details>

<details>
<summary><strong>Nothing happens / a Java error appears when launching</strong></summary>

Verify a compatible Java runtime is installed and active:

```bash
java -version          # should report 17 or newer
archlinux-java status  # list and switch between installed JREs
```

If needed, set the default: `sudo archlinux-java set java-17-openjdk`.
</details>

<details>
<summary><strong>The icon or menu entry doesn't appear</strong></summary>

Refresh the desktop caches, then log out and back in:

```bash
sudo update-desktop-database
sudo gtk-update-icon-cache /usr/share/icons/hicolor
```
</details>

## Contributing

Issues and pull requests are welcome — for example, keeping the bundled JAR current, improving the desktop integration, or refining the `PKGBUILD`. Please keep changes focused on the packaging; anything relating to the launcher itself should be reported to the [upstream project](https://llaun.ch/).

## License

The packaging files in this repository (PKGBUILD, wrapper script, desktop entry, README, and related metadata) are provided under the [MIT License](LICENSE).

The **Legacy Launcher** application, its name, logo, and JAR file are the property of their respective authors and are distributed under their own license and terms. See [llaun.ch](https://llaun.ch/) for details.
