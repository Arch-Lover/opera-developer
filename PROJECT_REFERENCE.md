# Opera Developer Arch Package — Project Reference

## Purpose

This repository packages the **Opera Developer** browser for Arch Linux. It
downloads Opera's upstream AMD64 Debian package, repackages its payload in the
Arch filesystem layout, and replaces Opera's bundled `libffmpeg.so` with the
prebuilt NW.js FFmpeg library.

The package currently targets `x86_64` only.

## Repository contents

| File | Role |
| --- | --- |
| `PKGBUILD` | Package metadata, sources, checksums, preparation, and install logic. |
| `opera` | Launcher template installed as `/usr/bin/opera-developer`. |
| `default` | System-wide launcher settings installed as `/etc/opera-developer/default`. |

`opera` and `default` contain `%pkgname%`/`%operabin%` placeholders. The
`prepare()` function substitutes them before packaging; do not replace them in
the tracked template files unless that behaviour is intentionally changing.

## Build and install

On an Arch-based system with the usual package build tools installed:

```sh
makepkg -si
```

For a build without installation:

```sh
makepkg
```

The resulting package archive is written to the repository directory. Build
downloads and extracted sources are placed in `src/` and package artifacts in
`pkg/`; these are normal `makepkg` working directories and should not be
committed.

## What the PKGBUILD does

1. Downloads the matching Opera Developer `.deb` from Opera.
2. Extracts `data.tar.xz` from the Debian archive.
3. Moves Opera from Debian's multiarch library path into
   `/usr/lib/opera-developer`.
4. Extracts the configured NW.js FFmpeg archive and installs its
   `libffmpeg.so` in Opera's library directory.
5. Sets the `opera_sandbox` binary setuid-root when it exists.
6. Installs the launcher and defaults file.
7. Rewrites Opera desktop-entry commands to use `opera-developer`.
8. Copies the upstream copyright file into Arch's package license location.

## Updating Opera Developer

1. Change `pkgver` in `PKGBUILD` to the desired upstream Opera Developer
   version. `pkgrel` normally remains `1` for a new upstream release.
2. Confirm the generated Opera URL resolves:

   `https://get.opera.com/pub/opera-developer/<pkgver>/linux/opera-developer_<pkgver>_amd64.deb`

3. Update the first value in `sha256sums` with the checksum of the new `.deb`.
4. Check whether the selected NW.js FFmpeg version remains ABI-compatible with
   the new Opera/Chromium version. If it changes, update
   `_nwjs_ffmpeg_version` and the fourth checksum.
5. Run `makepkg -s` (or `makepkg -si`) and launch the package to smoke-test it.

`makepkg -g` can generate fresh source checksums, but review the generated
values and keep the four entries in the same order as `source`.

## Runtime configuration

The launcher sources `/etc/opera-developer/default` when present. Set
`OPERA_FLAGS` there for system-wide command-line options. A user can override
those options for one launch or in their shell environment with
`OPERA_USER_FLAGS`:

```sh
OPERA_USER_FLAGS='--some-flag' opera-developer
```

The launcher executes the packaged browser binary at
`/usr/lib/opera-developer/opera-developer`.

## Maintenance notes

- The FFmpeg replacement is deliberate; preserve it unless the package policy
  changes.
- `backup=('etc/opera-developer/default')` preserves locally edited defaults
  across package upgrades using pacman's standard `.pacnew` behavior.
- The installed sandbox needs setuid permissions for Chromium-style sandboxing;
  avoid dropping the `chmod 4755` step without testing sandboxed launches.
- The package declares a conflict/replacement for
  `opera-developer-ffmpeg-codecs-bin`; installing this package should be the
  single source of Opera Developer FFmpeg codecs.
