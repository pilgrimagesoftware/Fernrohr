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
- [x] 4.2 Add a Linux install section to the App README covering:
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

## Notes

- Implemented in pilgrimagesoftware/Fernrohr-App#136, one commit per section 1-4. Tasks 1.1, 1.2,
  2.1, 3.1 and 4.1 are verified by `package.yml`'s own checks. They stay unticked until a package
  run on the branch passes. That run is dispatched without a tag, so nothing is published, and is
  queued behind GitHub's runner outage.
- GPUI set no window app id, so there was no Wayland `app_id` or X11 `WM_CLASS` for
  `StartupWMClass` to match. Every window now opens with app id `fernrohr` (`consts::APP_ID`). A
  unit test keeps the desktop entry's `StartupWMClass` equal to it.
- The icon also ships as `/usr/share/pixmaps/fernrohr.png`, in both the deb and the rpm. At 1254px
  it fits no hicolor size directory, so the icon theme would never find the copy cargo-packager
  installs there.
- cargo-packager has no `Recommends`, so the Linux leg adds it to the built deb (`dpkg-deb -R`,
  edit, `dpkg-deb --root-owner-group -b`) and reads it back with `dpkg-deb -f`, as design decision
  1 allows.
- **Deviation from decision 4:** the `checksums` job runs on every package run and keeps
  `SHA256SUMS` as an artifact. It attaches the file to the release only for a tag. That makes a
  develop run the dry run task 4.1 asks for, with no test tag or prerelease on the public repo.
- Running the checksum script locally caught a bug before CI ran it: redirecting `sha256sum`'s
  output into the package directory made `SHA256SUMS` list itself. It's written outside the
  directory, then moved in.
- The spec's "release not marked as published with a partial set" is only partly met. Each leg
  still attaches its own packages to a release that the release workflow has already published,
  as before. A failed leg does fail the run and stops `SHA256SUMS` from being attached, so a
  partial release is visible. Holding the release as a draft until every leg finishes would mean
  changing the shared `rust-release` workflow.
