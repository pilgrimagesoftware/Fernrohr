# Spec Delta

## ADDED Requirements

### Requirement: Pod detail links the objects the pod references
The pod detail panel SHALL show, as `resource-links` references, the pod's namespace, each of its
owners, its node, its service account, every ConfigMap, Secret and PersistentVolumeClaim its
volumes name, its image pull secrets, and every ConfigMap and Secret its containers read through
`envFrom` or `valueFrom`.

#### Scenario: Each owner is its own link
- **WHEN** a pod has more than one owner reference
- **THEN** each owner is shown as its own reference, not one combined line of text

#### Scenario: Volume sources are references
- **WHEN** a pod mounts a volume backed by a ConfigMap, a Secret or a PersistentVolumeClaim
- **THEN** that ConfigMap, Secret or claim is shown as a reference in the volume's entry

#### Scenario: Container environment sources are references
- **WHEN** a container reads environment variables from a ConfigMap or Secret, whole
  (`envFrom`) or by key (`valueFrom`)
- **THEN** that container's entry shows each such ConfigMap or Secret as a reference, once
  per object even when several keys come from it

#### Scenario: Image pull secrets are references
- **WHEN** a pod names image pull secrets
- **THEN** each is shown as a reference in the pod's fields

#### Scenario: Following the node link from a pod
- **WHEN** the application can display Nodes and the user follows a pod's node link
- **THEN** that node's panel opens (or is focused) in the pod's cluster context
