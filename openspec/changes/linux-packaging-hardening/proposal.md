# Proposal

## Why

Fernrohr already publishes a `.deb` and an AppImage for x86_64 Linux on every release, but those
packages are thin:

- The `.deb` declares only `libsecret-1-0`, even though the app runs `ssh` for tunnels and needs a
  running Secret Service for stored credentials. A fresh install can fail at the first tunnel, or
  at the first stored secret, with no hint about what is missing.
- Releases publish no checksums, so a download can't be verified.
- There is no `.rpm` for Fedora- and RHEL-family systems.
- There is no arm64 Linux build.

## What Changes

- Declare the real runtime needs in the `.deb`:
  - `Depends` on `openssh-client` and `libsecret-1-0`.
  - `Recommends` a Secret Service provider (`gnome-keyring | kwalletmanager | keepassxc`).
- Build an `.rpm` beside the `.deb` and AppImage, with equivalent `Requires` / `Recommends`.
- Add an aarch64 Linux leg, producing `.deb`, `.rpm` and AppImage.
- Publish a `SHA256SUMS` file with every release, covering every uploaded package from every
  platform.
- Document installing on Linux in the App README: which package to choose, what the
  dependencies are for, and how to verify a download against `SHA256SUMS`.

## Capabilities

### New Capabilities

- `release-packaging`: what a Fernrohr release publishes for each platform, the runtime
  dependencies its Linux packages declare, and the checksums that accompany it.

### Modified Capabilities

(none)

## Impact

- `App/app/Cargo.toml`: `[package.metadata.packager.deb]` dependencies, plus new RPM metadata
  (`[package.metadata.generate-rpm]`).
- `App/.github/workflows/package.yml`: an aarch64 Linux matrix leg on an arm64 runner, an RPM step
  using `cargo-generate-rpm` (cargo-packager has no RPM backend), and a checksum job that runs
  after every platform leg finishes.
- `App/.github/workflows/release.yml` / `tag-release.yml`, if the release upload lives there.
- `App/README.md`: a Linux install section.
