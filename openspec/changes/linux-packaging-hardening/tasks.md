# Tasks

## 1. Desktop entry and dependencies

- [ ] 1.1 Check in `app/assets/linux/fernrohr.desktop` (Name, Exec, Icon,
      `Categories=Development;Utility;`, `StartupWMClass` matching GPUI's app id), and point
      cargo-packager at it. Verify that `desktop-file-validate` passes in the Linux leg.
- [ ] 1.2 Set the `.deb` to depend on `openssh-client` and `libsecret-1-0`, and recommend
      `gnome-keyring | kwalletmanager | keepassxc`. Use a post-packaging control edit if
      cargo-packager can't express Recommends. Verify with a workflow step that asserts the result
      using `dpkg-deb -f <deb> Depends Recommends`.

## 2. RPM

- [ ] 2.1 Add `[package.metadata.generate-rpm]`: the binary, the desktop file, the icon, the
      licenses, `requires` (`openssh-clients`, `libsecret`) and `recommends` (`gnome-keyring`).
      Run `cargo generate-rpm` in each Linux leg. Verify with a workflow step that asserts the
      result using `rpm -qpR` and `rpm -qp --recommends`. Upload the `.rpm` as a build artifact and
      to tagged releases.

## 3. aarch64 Linux

- [ ] 3.1 Add the `ubuntu-24.04-arm` / `aarch64-unknown-linux-gnu` matrix leg, and generalize the
      Linux-only conditions. Verify that a `develop` run produces `.deb`, `.rpm` and AppImage
      artifacts for both architectures.

## 4. Checksums and documentation

- [ ] 4.1 Add the release-gated `checksums` fan-in job, which writes and uploads `SHA256SUMS` over
      every artifact. Verify with a dry run on a test tag or prerelease: `sha256sum --check
      SHA256SUMS` passes over the downloaded set, and there is one line per artifact.
- [ ] 4.2 Add a Linux install section to the App README covering:
      - choosing the `.deb`, `.rpm` or AppImage
      - what `openssh-client` and the keyring are for
      - the AppImage's host requirements
      - verifying downloads with `sha256sum --check --ignore-missing SHA256SUMS`

      Verify that the commands run as written.

## 5. Verification

- [ ] 5.1 Manual check on a real Linux machine, needing user confirmation:
      - Install the x86_64 `.deb` with `apt`, and confirm `openssh-client` is pulled in.
      - Launch from the desktop menu, and confirm the entry's name and icon look right and the
        window groups under the entry (`StartupWMClass`).
      - Store a tunnel secret, and confirm it lands in the keyring.
      - Run the AppImage on the same machine.
