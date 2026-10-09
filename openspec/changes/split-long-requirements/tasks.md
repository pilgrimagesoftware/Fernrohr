# Tasks

## 1. Split group A

- [ ] 1.1 Split the over-long requirements in `resource-browser` (7), `app-shell` (3), `command-system` (3), `pod-detail` (2), and `exec-terminal` (1); verify `openspec validate split-long-requirements --strict` passes and each MODIFIED block keeps its header and every scenario.

## 2. Split group B

- [ ] 2.1 Split the over-long requirements in `saved-panel-layouts` (5), `namespace-sets` (3), `cluster-connection` (2), `context-tunnel-binding` (2), `tunnel-config` (2), `managed-forward` (1), `pod-logs` (1), and `port-forward-indicators` (1); verify `openspec validate split-long-requirements --strict` passes and each MODIFIED block keeps its header and every scenario.

## 3. Integration

- [ ] 3.1 Archive the change and verify `openspec validate --specs --strict` passes for every spec.
