# live-busybox: staging

**Temporary branch.** It holds signed source and binary packages for
testing and for upload. It may be removed or rewritten at any time; the
package source lives on the `main` branch.

Each `deb-src/<version>/` folder holds:

| File | What it is |
|---|---|
| `*.dsc`, `*.debian.tar.xz`, `*.orig.tar.bz2`, `*.orig.tar.bz2.asc` | the signed source package |
| `*_amd64.deb`, `*_i386.deb` | binary packages |
| `*.buildinfo` | signed build information |
| `*_multi.changes` | signed upload file (source plus both architectures) |
| `SHA256SUMS` | checksums of all files in the folder |

Check a downloaded folder:

```sh
sha256sum -c SHA256SUMS
gpg --verify live-busybox_<version>_multi.changes
dscverify live-busybox_<version>.dsc
```
