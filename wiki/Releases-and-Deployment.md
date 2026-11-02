# Releases and Deployment

## CI pipeline

Two workflows build GuppyScreen, both inside the public `ballaswag/guppydev:latest` toolchain
container ([`docker/Dockerfile`](https://github.com/alexholco/openke/blob/main/docker/Dockerfile)
reproduces it from source). Each checks out submodules, applies the three patches in `patches/`
(lv_drivers, spdlog, lvgl), builds the bundled libraries, builds GuppyScreen, then packages each
asset with `scripts/release.sh`.

- **`pull_request.yml`** runs on every pull request into `main`. It builds all four variants
  (KE small screen, standard screen, Z-Bolt icons, aarch64) as a compile check and uploads them as
  workflow artifacts. Nothing is published.
- **`build.yml`** runs only when a pull request is **merged** into `main`. It builds the merge commit
  and publishes a GitHub release (see below).

Nothing is built on plain pushes, and there are no nightly releases.

### Build output

This is a KE-focused fork, so releases carry **only the Ender-3 V3 KE asset**:

| Asset | Toolchain | Theme | Small screen | Notes |
|---|---|---|---|---|
| `guppyscreen-smallscreen.tar.gz` | `mipsel-buildroot-linux-musl-` | material | **yes** | **The Ender-3 V3 KE asset** |

- `GUPPYSCREEN_VERSION` is `pr<N>-<sha7>`: the merged PR's number and the merge commit.
- The release is tagged `pr<N>` and named after the PR.

## Packaging locally

After a successful build (see [Building from Source](Building-from-Source)), package a tarball with:

```bash
GUPPYSCREEN_VERSION=0.1.1-ke-gui-fixes GUPPY_THEME=blue \
  bash scripts/release.sh guppyscreen-smallscreen
```

`scripts/release.sh`:

1. Strips the binary (using `$CROSS_COMPILE`strip).
2. Copies the binary, `k1/k1_mods`, `k1/scripts`, `themes/`, the installers (`installer.sh`,
   `installer-deb.sh`), `update.sh`, and `debian/` into `releases/guppyscreen/`.
3. Writes a `.version` file (`{version, theme, asset_name}`).
4. Produces `<asset>.tar.gz`.

The on-printer installer downloads the matching `*.tar.gz` from the GitHub release.

## Release tags

Each merged pull request publishes one release tagged `pr<N>`. PR numbers don't follow merge order
(a PR opened earlier can be merged later), so "newest" means most recently **published**, which is
what GitHub's latest-release API returns. Both the installer and the on-screen updater (`update.sh`)
use that API; the updater installs the latest release whenever its tag differs from the installed
version's `pr<N>` prefix. See the [Releases page](https://github.com/alexholco/openke/releases).

Older tags from before this fork (`vX.Y.Z-OpenKE`, `v0.1.1-ke-gui-fixes`, the `0.0.x-beta`
series) remain in the repository history.

### Installing a specific build

Pin any release tag with `PINNED_RELEASE`:

```sh
PINNED_RELEASE=pr4 sh -c "$(curl -sL https://raw.githubusercontent.com/alexholco/openke/main/scripts/installer.sh)"
```

(Use `curl`, not the `wget` form shown elsewhere in this wiki, if your printer's `wget` can't
complete a TLS handshake against GitHub — see [Troubleshooting](Troubleshooting).) Re-run the
normal install command with no `PINNED_RELEASE` to go back to the newest release.

See **[Troubleshooting](Troubleshooting)** for current installer/upgrade caveats.
