# Tasks

## 1. Desktop entry and dependencies

- [x] 1.1 Check in `app/assets/linux/fernrohr.desktop` (Name, Exec, Icon,
      `Categories=Development;Utility;`, `StartupWMClass` matching GPUI's app id), and point
      cargo-packager at it. Verify that `desktop-file-validate` passes in the Linux leg.
- [x] 1.2 Set the `.deb` to depend on `openssh-client` and `libsecret-1-0`, and recommend
      `gnome-keyring | kwalletmanager | keepassxc`. Use a post-packaging control edit if
      cargo-packager can't express Recommends. Verify with a workflow step that asserts the result
      using `dpkg-deb -f <deb> Depends Recommends`.

## 2. RPM

- [x] 2.1 Add `[package.metadata.generate-rpm]`: the binary, the desktop file, the icon, the
      licenses, `requires` (`openssh-clients`, `libsecret`) and `recommends` (`gnome-keyring`).
      Run `cargo generate-rpm` in each Linux leg. Verify with a workflow step that asserts the
      result using `rpm -qpR` and `rpm -qp --recommends`. Upload the `.rpm` as a build artifact and
      to tagged releases.

## 3. aarch64 Linux

- [x] 3.1 Add the `ubuntu-24.04-arm` / `aarch64-unknown-linux-gnu` matrix leg, and generalize the
      Linux-only conditions. Verify that a `develop` run produces `.deb`, `.rpm` and AppImage
      artifacts for both architectures.

## 4. Checksums and documentation

- [x] 4.1 Add the release-gated `checksums` fan-in job, which writes and uploads `SHA256SUMS` over
      every artifact. Verify with a dry run on a test tag or prerelease: `sha256sum --check
      SHA256SUMS` passes over the downloaded set, and there is one line per artifact.
- [x] 4.2 Add a Linux install section to the App README covering:
      - choosing the `.deb`, `.rpm` or AppImage
      - what `openssh-client` and the keyring are for
      - the AppImage's host requirements
      - verifying downloads with `sha256sum --check --ignore-missing SHA256SUMS`

      Verify that the commands run as written.

## 5. Verification

- [x] 5.1 Manual check on a real Linux machine, needing user confirmation:
      - Install the x86_64 `.deb` with `apt`, and confirm `openssh-client` is pulled in.
      - Launch from the desktop menu, and confirm the entry's name and icon look right and the
        window groups under the entry (`StartupWMClass`).
      - Store a tunnel secret, and confirm it lands in the keyring.
      - Run the AppImage on the same machine.

## Notes

- Implemented in pilgrimagesoftware/Fernrohr-App#136, one commit per section 1-4. Tasks 1.1, 1.2,
  2.1, 3.1 and 4.1 were verified by a package run on the branch, dispatched without a tag so nothing
  was published (Fernrohr-App Actions run 37374655259):
  - all three legs passed their deb, rpm, desktop-entry and icon checks
  - it produced a `.deb`, `.rpm` and AppImage for x86_64 and aarch64, plus the dmg
  - `checksums` wrote `SHA256SUMS` with one line for each of the 7 packages, and
    `sha256sum --check` passed over the downloaded set
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
- The spec now says what happens when a leg fails. The run fails and `SHA256SUMS` isn't attached,
  but legs that succeeded may already have attached their packages. Draft-release gating is
  deferred (design.md decision 4) because it needs a change to the shared `rust-release` workflow.
- The icon is also installed at 256x256 and 512x512 under `/usr/share/icons/hicolor`. Those
  scalings are checked in under `app/assets/linux/icons`, and the pixmaps copy stays as a
  fallback.
