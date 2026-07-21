# Plan: fetch the JAR at build time + packaging/security fixes

Working doc for `legacy-launcher` PKGBUILD. All endpoint claims below were verified
against the live hosts and against the upstream `.deb` on 2026-07-21.

---

## 1. Feature: download the JAR instead of vendoring it

### Today

`LegacyLauncher.jar` (11 MB) is committed to the repo and listed as a local `source=()`
entry with `sha256sums=('SKIP')`. Updating means manually downloading the jar, committing
a new 11 MB blob, and rebuilding.

### Change

Fetch from upstream at build time so `makepkg` retrieves and verifies the jar, and drop
the blob from git.

```bash
pkgver=1.40.3
source=("LegacyLauncher.jar::https://llaun.ch/jar"
        'legacy-launcher.svg'
        'legacy-launcher_256.png'
        'legacy-launcher_512.png')
noextract=('LegacyLauncher.jar')
sha256sums=('09ba7517c8a9b30780ff9a6de230b8905247835ae0da914d66991e2b3b4b6c4e'
            'SKIP' 'SKIP' 'SKIP')
```

Then:

```bash
git rm --cached LegacyLauncher.jar
echo 'LegacyLauncher.jar' >> .gitignore
```

### Why `https://llaun.ch/jar` and not `dl.llaun.ch/legacy/bootstrap`

Both serve byte-identical files (both hash to `09ba7517…4b6c4e`). `llaun.ch/jar` is
preferred because:

- It is the endpoint upstream's own `.deb` `postinst` targets, so it is the least likely
  to be moved or renamed.
- It is mirrored. `postinst` falls back across `llaun.ch`, `eu1.llaun.ch`, `lln4.ru`,
  `ru1.lln4.ru`; all four resolve to the same artifact.
- It is not behind a Cloudflare challenge (see §4).

### Two gotchas that will bite

**`makepkg` ignores `content-disposition`.** It names the file from the URL basename, so
the download would land as `jar`. The `filename::url` rename syntax above is required.

**A `.jar` is a zip and `makepkg` will unpack it.** `file --mime-type` reports
`application/zip`, and makepkg's `source/file.sh` matches `application/zip` for
extraction. Without `noextract`, `$srcdir` gets an exploded `META-INF/` and no jar.

### `pkgver`

Current `1.0` is invented. The real version is in the jar at `META-INF/bootstrap-meta.json`:

```json
{"version":"1.40.3+legacy","shortBrand":"legacy","brand":"Stable"}
```

Use `1.40.3`. Do **not** take the version from the `.deb` — its control file says
`Version: 1.0` with a 2024-02-28 build date and is stale.

---

## 2. Fixes to carry over from the upstream `.deb`

### 2a. `StartupWMClass` (the icon bug)

The desktop entry has no `StartupWMClass`, so the running window does not associate with
its launcher icon — the taskbar/dock shows a generic or duplicate entry. Upstream sets:

```
StartupWMClass=net-legacylauncher-LegacyLauncher
```

Add this to the `.desktop` heredoc in `package()`.

### 2b. The 256×256 icon is dead weight

`legacy-launcher_256.png` is committed to the repo but appears in neither `source=()` nor
`package()` — it is never installed. Either install it to
`/usr/share/icons/hicolor/256x256/apps/legacy-launcher.png` (upstream ships 128 and 256)
or delete it from the repo. Installing it is the better call; 256 is a very common tray
and dock size, and shipping only 512 forces downscaling.

### 2c. Desktop entry polish

Worth adopting from upstream's entry:

- `Keywords=minecraft` — makes the app findable by what users actually type.
- A real `Comment` (the current one is fine, but upstream also ships `Comment[ru]`).

Skip upstream's `Version=1.0` field — that is the *desktop entry spec* version, not the
app version, and `1.0` is correct there. Leave it.

---

## 3. What NOT to copy from the `.deb`

Upstream downloads the jar in `postinst`, **after** unpacking:

```bash
for host in llaun.ch eu1.llaun.ch lln4.ru ru1.lln4.ru
do
    wget https://$host/jar -O "$SHARED_BOOTSTRAP" && break
done
```

Arch has an equivalent (`.install` + `post_install()`), but this pattern should not be
ported:

- **Files created in `post_install()` are not owned by the package.** `pacman -R` would
  leave the jar orphaned in `/usr/lib`, breaking the clean-removal property the README
  advertises.
- **It breaks offline, chroot, and CI builds**, and makes the package non-reproducible.
- **It moves the payload outside `sha256sums`**, discarding the only integrity check
  available.

Upstream's version also has a genuine bug worth not inheriting: `wget -O "$FILE"` creates
the file before the fetch is known to succeed, so a failed download still leaves a
truncated or empty file behind. The guard that follows —

```bash
if [ ! -f "$SHARED_BOOTSTRAP" ]
```

— can therefore effectively never fire. The failure mode is an install that reports
success and produces a launcher that dies at runtime.

`source=()` + a pinned hash beats this on every axis: makepkg fetches, pacman tracks the
file, and the checksum is enforced before `package()` ever runs.

---

## 4. Security upgrades

### 4a. Pin the hash — never `SKIP` on the remote source

This is the single most important item. Because the endpoint sits behind Cloudflare, an
unpinned remote source can silently bake a challenge page into the package.

Confirmed: every other download path on `dl.llaun.ch` (`deb`, `portable`, `source`,
`installer`, `dmg`) returns HTTP 403 with `cf-mitigated: challenge` and a ~5.6 KB
"Just a moment…" body. A browser user agent does not help. With `SKIP`, curl saves that
HTML, `package()` installs it as `LegacyLauncher.jar`, and the build "succeeds".

A pinned hash turns that failure into a loud checksum mismatch at fetch time.

### 4b. Know what the hash does and does not cover

The shipped file is a **bootstrap**, not the launcher:

```
Start-Class: net.legacylauncher.bootstrap.BootstrapStarter
```

It fetches the actual launcher at runtime into the user's home directory
(`~/.tlauncher/bin` per upstream's wrapper). So the pin verifies *the downloader*; the
real payload arrives later, entirely outside pacman's control. **No PKGBUILD change can
raise this ceiling** — it is inherent to how upstream ships. It should be disclosed in
the README rather than papered over.

### 4c. There is no signature to verify

Checked and confirmed absent:

- No `META-INF/*.SF`, `*.RSA`, `*.DSA` — the jar is **not** JAR-signed.
- No detached `.asc`/`.sig` and no published checksums file.

So `validpgpkeys=()` / `source=(...{,.sig})` is not an option here, and the sha256 pin is
the *only* integrity mechanism available. Worth stating explicitly so a future maintainer
does not assume signature verification is happening.

### 4d. The download crosses domains via redirect

`https://llaun.ch/jar` is a 301 → `lln4.cc/jar` → 302 → final 200. The chain leaves the
original domain. This is normal for their CDN, and the hash pin fully mitigates it, but
it is another reason `SKIP` is unacceptable: without the pin, trust extends to every host
in that chain.

### 4e. Optional: sanity-check the artifact in `prepare()`

Largely redundant once 4a is in place, but useful for a maintainer who temporarily uses
`SKIP` while bumping versions:

```bash
prepare() {
    # Fail loudly if we were served an HTML challenge page instead of a JAR
    bsdtar -tf LegacyLauncher.jar >/dev/null 2>&1 \
        || { echo "ERROR: LegacyLauncher.jar is not a valid archive" >&2; return 1; }
}
```

### 4f. Harden the wrapper script

Current wrapper is a single `exec java -jar …` line, which is already sound (`exec` +
`"$@"` quoting is correct). Two small improvements:

- Add `set -eu` for consistency, even though `exec` makes it near-cosmetic.
- Consider `exec java -jar /opt/legacy-launcher/LegacyLauncher.jar "$@"` staying as-is
  rather than adopting upstream's copy-to-`$HOME` dance — ours is simpler and keeps the
  package-owned jar authoritative.

---

## 5. Maintenance procedure (README update)

The URL is a rolling "latest" endpoint with no version in the path, so a pinned hash will
start failing the moment upstream publishes a new build. That is the intended signal, not
a defect. Replace the current "Updating Legacy Launcher" section with:

```bash
updpkgsums                       # refresh sha256sums from upstream
# read the new version from META-INF/bootstrap-meta.json, bump pkgver
makepkg -si
```

Also note in the README that pacman manages the bootstrap only, and that the launcher
self-updates into `$HOME` outside package management (§4b).

---

## 6. Order of work

1. `source=()` → remote URL, add `noextract`, pin sha256, bump `pkgver` to 1.40.3.
2. `git rm --cached` the jar, add to `.gitignore`.
3. Add `StartupWMClass` + `Keywords` to the desktop entry.
4. Install the 256×256 icon (or drop the file).
5. Optional `prepare()` sanity check; `set -eu` in the wrapper.
6. README: new update procedure, self-update disclosure, repo-structure table fixup
   (it currently lists `LegacyLauncher.jar` as a repo file).

## 7. Verification after the change

```bash
makepkg -f                                  # must fetch + verify, not extract the jar
bsdtar -tf legacy-launcher-1.40.3-1-any.pkg.tar.zst | grep -E 'jar|desktop|icons'
sudo pacman -U legacy-launcher-1.40.3-1-any.pkg.tar.zst
legacy-launcher --help
pacman -Ql legacy-launcher                  # confirm every installed file is tracked
sudo pacman -R legacy-launcher && ls /opt/legacy-launcher 2>&1   # must be gone
```
