# Legacy Launcher PKGBUILD

An unofficial Arch Linux package for installing **Legacy Launcher** as a native application.

This package installs the official Legacy Launcher JAR under `/opt`, creates a launcher script, installs a desktop entry, and registers application icons so it integrates with your desktop environment.

> **Note**
>
> This repository is **not affiliated with the Legacy Launcher project**. It simply packages the official launcher for Arch Linux.

---

## Features

* Native Arch package (`makepkg`)
* Desktop menu integration
* SVG and PNG application icons
* Launcher command available as `legacy-launcher`
* Installs to standard Linux locations

---

## Requirements

* Arch Linux (or an Arch-based distribution)
* `base-devel`
* Java 17 or newer

Install the required packages:

```bash
sudo pacman -S --needed base-devel jre17-openjdk
```

---

## Repository Structure

```
.
├── PKGBUILD
├── LegacyLauncher.jar
├── legacy-launcher.svg
├── legacy-launcher_512.png
└── README.md
```

---

## Installation

Clone the repository:

```bash
git clone <repository-url>
cd <repository-name>
```

Build and install the package:

```bash
makepkg -si
```

Once installed, you can launch it from:

* Your application launcher
* The command line:

```bash
legacy-launcher
```

---

## Updating Legacy Launcher

When a new version of Legacy Launcher is released:

1. Download the latest **LegacyLauncher.jar** from the official Legacy Launcher website.
2. Replace the existing `LegacyLauncher.jar` in this repository with the new one.
3. Rebuild and reinstall the package:

```bash
makepkg -si
```

The new launcher will overwrite the previous installation.

---

## Uninstall

Remove the package using pacman:

```bash
sudo pacman -R legacy-launcher
```

---

## License

The packaging files in this repository are provided under the MIT License (or your preferred license).

The **Legacy Launcher** application, its name, logo, and JAR file are the property of their respective authors and are distributed under their own license and terms.
