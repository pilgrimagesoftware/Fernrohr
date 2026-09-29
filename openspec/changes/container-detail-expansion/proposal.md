# Proposal

## Why

The Containers section's cards (`openspec/changes/pod-detail-panel`) show a fixed summary - image,
ready state, restart count, ports, resource requests/limits. That's deliberately what fits on a
card among several containers, but a user investigating one specific container (why it's
restarting, what its full env/volume mounts are) has nowhere to go from there. The `Container`
spec object carries real detail beyond the summary - environment variables, volume mounts, probes,
security context, command/args - none of which the summary shows or has anywhere to show.

## What Changes

- Each container card becomes selectable/expandable, revealing the fields the summary omits:
  environment variables (names only for secret/configmap-sourced values, not their resolved
  contents - see design.md on why), volume mounts, liveness/readiness/startup probes,
  command/args, security context.
- The summary card's existing fields (image, ready, restarts, ports, requests/limits) stay exactly
  as they are today; expansion adds, it doesn't replace.

## Capabilities

### Modified Capabilities
- `pod-detail`: containers can be expanded for detail beyond the summary card.

## Impact

- `app/src/k8s/resource/pod_detail.rs`: `ContainerSummary` gains the additional fields (or a
  separate `ContainerDetail` fetched/computed only on expansion, if the full field set is large
  enough to not want it computed unconditionally for every container - a design.md decision), and
  `PodFieldValue::Containers`' render gains an expand/collapse affordance per card.
