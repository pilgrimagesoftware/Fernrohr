# Design

## Context

`package.yml` runs a two-leg matrix:

- macOS `aarch64-apple-darwin` producing `app,dmg`
- Linux `x86_64-unknown-linux-gnu` producing `appimage,deb` on `ubuntu-latest`

Both legs use `cargo packager`. For tagged releases each leg runs `gh release upload`. The Linux
leg installs the GUI build dependencies with apt. `[package.metadata.packager.deb]` sets
`depends = ["libsecret-1-0"]`, and the AppImage bundles `libsecret-1.so.0`. The sibling project
dtrpg-app.rs follows the same pattern and has the same gaps, so there's nothing further to copy
from it.

## Goals / Non-Goals

**Goals:**

- Correct Linux dependency metadata.
- RPM output.
- An aarch64 Linux build.
- Checksums for every release artifact.

**Non-Goals:**

- Signing Linux packages (GPG-signed repositories, `.rpm` signatures), a hosted apt or dnf
  repository, Flatpak, Snap, or AUR. Each is a separate decision.
- Windows.
- Verifying the generated `.desktop` entry and running on real Linux hardware. The user will do
  that by hand (task 5.1).

## Decisions

### 1. Dependency declarations

- **`.deb`:**
  - `depends = ["openssh-client", "libsecret-1-0"]`
  - Recommends `gnome-keyring | kwalletmanager | keepassxc`
- **`.rpm`:**
  - `requires = { openssh-clients = "*", libsecret = "*" }`
  - `recommends = { gnome-keyring = "*" }`

If cargo-packager's deb config can't express `Recommends`, add the field after packaging: unpack
with `dpkg-deb -R`, edit `DEBIAN/control`, and rebuild with `dpkg-deb -b`. Do this in a small
workflow step with a test that reads the result back with `dpkg-deb -f`, rather than giving up the
recommendation.

### 2. RPM via `cargo-generate-rpm`

cargo-packager has no RPM backend. Add `[package.metadata.generate-rpm]` to `app/Cargo.toml`,
covering:

- assets: the binary into `/usr/bin`, the `.desktop` file, the icon, and the license files that
  cargo-packager's `resources` already ship
- requires and recommends, as in decision 1

Run `cargo generate-rpm --target <triple>` after `cargo packager` on each Linux leg, reusing its
release build.

To keep the `.desktop` file identical across formats, check one in at
`app/assets/linux/fernrohr.desktop` and point both cargo-packager (`desktop-template`, or
`files`) and generate-rpm at it, rather than relying on cargo-packager's generated copy. It sets:

- `Name`, `Exec`, `Icon`
- `Categories=Development;Utility;`
- `StartupWMClass=` matching GPUI's app id

### 3. aarch64 Linux leg

Add a matrix entry `{ name: Linux-arm64, os: ubuntu-24.04-arm, target:
aarch64-unknown-linux-gnu, formats: appimage,deb }`. It builds natively on GitHub's arm64 runner,
with no cross-compilation, so the apt dependency list and the gnu linkage stay the same. Change
the Linux-only conditions (`if: matrix.name == 'Linux'`) to `startsWith(matrix.name, 'Linux')`.

### 4. Checksums as a fan-in job

Add a `checksums` job with `needs:` set to the package matrix. It runs on every package run, so a
`develop` or dispatched run is the dry run, with no test tag on the public repo, and attaches its
result to the release only for a tag. It:

1. downloads every leg's artifacts
2. runs `sha256sum` over all `.dmg`, `.deb`, `.rpm` and AppImage files, file names only, no paths
3. writes `SHA256SUMS`
4. keeps `SHA256SUMS` as a run artifact, and for a tag uploads it to the release

Because the job needs every leg, a failed leg stops `SHA256SUMS` from being published, and the run
fails. Legs that succeeded have already attached their own packages, though: each leg uploads to
a release the release workflow has already published.

- **Deferred: draft-release gating.** Holding the release as a draft until every leg and
  `SHA256SUMS` are attached, then publishing it, would stop a partial release from ever being
  visible. It needs a change to the shared `rust-release` workflow, which creates and publishes
  the release, so it is left for a separate change.

- **Alternative: each leg appends its own checksums.** Rejected, because concurrent uploads to one
  file race, and per-leg files aren't what `sha256sum --check` users expect.

## Risks / Trade-offs

- [A dependency's package name differs across distros, e.g. `openssh-clients` on Fedora versus
  `openssh-client` on Debian] → the names are per format, and the Linux test on the user's machine
  (5.1) checks the deb path on a real system.
- [The arm64 runner image lacks a package] → the leg fails visibly, and the apt list is shared, so
  any fix applies to both architectures.
- [An AppImage can't declare dependencies] → the README documents the host requirements.
