# live-busybox

Static BusyBox for the MX Linux / antiX live-initrd.

`live-busybox` is a Debian source package that builds BusyBox from
**Debian's busybox source** (1:1.38.0-3, with all Debian patches) as one
small, **statically linked (musl)** binary for the live-initrd
(`mx-iso-template`).

It replaces the old BusyBox 1.31.1 that had been carried in the
live-initrd as a prebuilt binary. The new binary is built reproducibly
from published source, for **amd64** and **i386**.

This repository is based on Debian's busybox git repository
(<https://salsa.debian.org/installer-team/busybox>). Debian's history,
patches and upstream imports are kept, and the live-busybox packaging
sits on top. Our complete change against Debian:

```sh
git diff 'debian/1%1.38.0-3' mx/1.38.0-3mx1
```

## What the package installs

The package is a build helper only. It installs into its own directory and
creates no applet symlinks, so it does not touch the system busybox.

| Path | Content |
|---|---|
| `/usr/lib/live-busybox/bin/busybox` | the static binary |
| `/usr/lib/live-busybox/busybox.links` | list of applet paths |
| `/usr/lib/live-busybox/busybox.config` | build configuration |

The configuration is also embedded in the binary (`busybox bbconfig`).

`mx-iso-template` copies the binary into the initrd at build time.

## How it differs from Debian's busybox

- **One flavour, "live":** `debian/config/pkg/live` has the same applet set
  as the old 1.31.1 initrd busybox, updated to 1.38.0.
- **Linked statically against musl** instead of glibc: about 616K instead
  of 1.7M.
- **musl limits:** `FEATURE_VI_REGEX_SEARCH` is off. Two testsuite cases
  that hit musl limits are skipped (`debian/testsuite-musl.diff`).
- **`live-ttyname-fallback-without-proc.patch`:** `tty` also works in the
  early initrd, before `/proc` is mounted.

See `debian/changelog` and `debian/README.Maintainer` for details.

## Branches and tags

| Branch | Content |
|---|---|
| `mx` (default) | live-busybox: Debian's tree plus our `debian/` |
| `master` | Debian's packaging, as on salsa |
| `upstream` | upstream imports, as on salsa |
| `pristine-tar` | data to recreate the exact upstream tarball |
| `staging` | temporary: signed source and binary packages in `deb-src/<version>/`. This branch may be removed at any time. |

| Tag | Meaning |
|---|---|
| `mx/<version>` | live-busybox releases, e.g. `mx/1.38.0-3mx1` |
| `debian/1%<version>` | Debian's uploads (`%` stands for the epoch `1:`) |
| `upstream/<version>` | upstream releases |

## Building

The source is kept as a `3.0 (quilt)` tree, with the patches in
`debian/patches` **not applied**.

**1. Get the orig tarball** and put it next to the tree. `pristine-tar`
recreates the byte-identical busybox.net tarball from git:

```sh
git branch pristine-tar origin/pristine-tar   # once, after cloning
git branch upstream origin/upstream           # once, after cloning
pristine-tar checkout ../live-busybox_1.38.0.orig.tar.bz2
```

Or let git-buildpackage do the orig tarball and the source package in
one step (needs a clean tree):

```sh
gbp buildpackage -S -us -uc
```

The upstream signature (`.orig.tar.bz2.asc`) is not stored in
pristine-tar. It is on the `staging` branch, or at busybox.net as
`busybox-1.38.0.tar.bz2.sig`.

Upstream signs with a DSA-1024/SHA-1 key, which sqv rejects. Check the
signature with gpgv instead:

```sh
gpg --dearmor < debian/upstream/signing-key.asc > ../bb-upstream.gpg
gpgv --keyring ../bb-upstream.gpg ../live-busybox_1.38.0.orig.tar.bz2.asc ../live-busybox_1.38.0.orig.tar.bz2
```

**2. Build the source package and the binaries** (sbuild, unshare mode):

```sh
dpkg-source -b .
dpkg-source --after-build .    # unapply the patches in the tree again
cd ..
sbuild --chroot-mode=unshare --dist=trixie --arch=amd64 live-busybox_<version>.dsc
sbuild --chroot-mode=unshare --dist=trixie --arch=i386  live-busybox_<version>.dsc
```

## License

BusyBox is GPL-2; see `debian/copyright`. The Debian packaging is by the
Debian busybox maintainers; MX changes are by fehlix.
