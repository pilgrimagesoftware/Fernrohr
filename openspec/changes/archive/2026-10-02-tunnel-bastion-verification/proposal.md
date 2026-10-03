## Why

`tunnel-subsystem` implements and unit-tests the full `ManagedForward` subsystem (SSH tunnels,
Kubernetes port-forwarding, tunnel config/secrets, context binding, and pause/resume on flap or
credential-plugin `401`) against fakes and local stand-ins (a throwaway local `sshd`, a hand-rolled
fake Kubernetes API server, a self-signed TLS spike). What it cannot exercise is the real thing: an
actual bastion host, a second OS, and a cluster whose kubeconfig uses an exec credential plugin. This
change is that end-to-end verification pass, split out so `tunnel-subsystem` can ship and be archived
on its own merits (all seven implementation sections done, 136/136 tests passing) without blocking on
infrastructure that doesn't exist yet.

This change depends on `tunnel-subsystem` and stays open, unscheduled, until a bastion host and a
credential-plugin-backed cluster are available to test against.

## What Changes

- No new code is anticipated; this is a verification pass against the `tunnel-subsystem`
  implementation. Any real bug it surfaces gets its own fix (in this change or a follow-up), but the
  four checks below are the deliverable.

## Impact

- No spec deltas expected. If verification finds a gap between spec and behavior, this change (or a
  fix change it spawns) corrects the implementation to match `tunnel-subsystem`'s existing specs
  rather than changing them.
- Blocked on infrastructure: a reachable SSH bastion (macOS and Linux client), and a cluster reachable
  only via an exec credential plugin (e.g. `aws eks get-token`) or a faithful stand-in.
