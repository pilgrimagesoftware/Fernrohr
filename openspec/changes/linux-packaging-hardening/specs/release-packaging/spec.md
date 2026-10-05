# Spec Delta

## Purpose

Defines what a Fernrohr release publishes for each platform: package formats and architectures,
the runtime dependencies the Linux packages declare, and the checksums that let a download be
verified.

## ADDED Requirements

### Requirement: Release artifacts per platform

Every tagged release SHALL publish:

- for macOS on Apple silicon: a `.dmg`
- for Linux on x86_64 and on aarch64: a `.deb`, an `.rpm`, and an AppImage

Each artifact's file name SHALL include the version and the architecture.

#### Scenario: Linux artifacts on a release

- **WHEN** a release tag is published
- **THEN** the release lists a `.deb`, an `.rpm`, and an AppImage for x86_64 and for aarch64, each named with the version and architecture

#### Scenario: A failed platform leg blocks publishing

- **WHEN** packaging fails for any platform leg of a tagged release
- **THEN** the workflow run fails, and the release is not marked as published with a partial set of artifacts

### Requirement: Linux packages declare their runtime dependencies

The `.deb` and `.rpm` SHALL require the SSH client and the Secret Service client library the
application uses at run time. They SHALL recommend, without requiring, at least one Secret Service
provider. The AppImage SHALL bundle the Secret Service client library, and its documentation SHALL
name the SSH client and a running Secret Service provider as host requirements.

#### Scenario: Installing the deb pulls in ssh

- **WHEN** the `.deb` is installed with `apt` on a system without an SSH client
- **THEN** `apt` installs the SSH client package as a dependency

#### Scenario: Keyring provider is recommended, not forced

- **WHEN** the `.deb` is installed on a system that already runs a different Secret Service provider
- **THEN** installation succeeds without installing another provider

#### Scenario: The rpm declares the same needs

- **WHEN** the `.rpm`'s metadata is inspected with `rpm -qpR` and `rpm -qp --recommends`
- **THEN** it requires the SSH client and the Secret Service client library, and recommends a Secret Service provider

### Requirement: Release checksums

Every tagged release SHALL publish a `SHA256SUMS` file listing the SHA-256 digest of every package
artifact on that release, in the format `sha256sum --check` accepts.

#### Scenario: Verifying a download

- **WHEN** a user downloads an artifact and `SHA256SUMS` into one directory and runs `sha256sum --check --ignore-missing SHA256SUMS`
- **THEN** the downloaded artifact is reported as `OK`

#### Scenario: Every artifact is covered

- **WHEN** a release is published
- **THEN** `SHA256SUMS` has exactly one line for each `.dmg`, `.deb`, `.rpm`, and AppImage on the release
