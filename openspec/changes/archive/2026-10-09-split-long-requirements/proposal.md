# Proposal

## Why

Thirteen main specs fail `openspec validate --specs --strict` because 33 requirements have
descriptions over 500 characters. Each bundles several behaviors into one block. That makes the
requirements harder to review and to test against, and it hides genuinely new strict failures
behind the existing ones.

## What Changes

- Split every over-long requirement in the affected specs. The original requirement keeps its
  header and every scenario, and its description is cut down to one behavior without changing
  its meaning. Each behavior removed from it becomes its own requirement with its own scenario.
- No behavior changes. No application code changes.

## Capabilities

### New Capabilities

- None.

### Modified Capabilities

- `app-shell`, `cluster-connection`, `command-system`, `context-tunnel-binding`, `exec-terminal`,
  `managed-forward`, `namespace-sets`, `pod-detail`, `pod-logs`, `port-forward-indicators`,
  `resource-browser`, `saved-panel-layouts`, `tunnel-config`: over-long requirements are split
  into single-behavior requirements. The behavior they specify is unchanged.

## Impact

- `openspec/specs/` only. After archiving, `openspec validate --specs --strict` passes for every
  spec.
