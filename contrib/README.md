The __contrib__ directory showcases use of various aurutils components, similar to the `EXAMPLES` section in `aur(1)`. Examples here do not necessarily fit into a single pipeline and are thus kept as separate programs.

To use a script with `aur(1)`, copy it to a directory in `PATH`. Alternatively, create a symbolic link if you have no local changes and always want to use the latest version.

```
$ cp -s /usr/share/aurutils/contrib/aur-vercmp-devel /usr/local/bin
$ aur vercmp-devel
```

### aur-vercmp-devel

This script takes the contents of a local repository (`aur sync --list`) and matches them against a common pattern for VCS packages (`grep -E $AURVCS`). It then looks in the `aur-sync` cache (`find $AURDEST`) for relevant directories. Any existing `PKGBUILD`s in these directories are executed, with upstream sources updated to their latest revision. (`aur srcver` using `makepkg -o`)

In the last line, the resulting package versions (`vcs_info`) are compared against the local repository (`db_info`). If the package version is newer, it is printed to `stdout`. This may be combined with `aur-sync` as follows:

```
aur vercmp-devel "$@" | cut -d: -f1 | aur sync --no-ver-shallow -
```

VCS packages typically have `pkgver` set to the upstream revision at the time of package submission, making the AUR-advertised version older than the latest version. Here, the `--no-ver-shallow` option ignores the AUR information _only_ for packages from standard input (`-`).

As described, the above relies on already available `PKGBUILD`s. If the `aur-sync` cache is sparse or the package has meanwhile been updated by the AUR maintainer (for example, to indicate a new upstream), information reported by `aur-vercmp-devel` may be incomplete. 

The following mediates this by downloading all VCS packages in a local repository anew, with all build files and their diffs offered for inspection.

```
aur sync --list | cut -f2 | grep -E "$AURVCS" | aur sync --no-ver --print -
```