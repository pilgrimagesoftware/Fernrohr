# Proposal

## Why

Fernrohr's panels and lists are all text: a Pods panel, a ConfigMap card and a Deployment link look
alike until you read them. A recognizable picture per kind makes panels, tabs and links scannable
at a glance, which matters most when a window holds many panels.

## What Changes

- **A full-colour icon per resource kind**, from the official Kubernetes community icon set (used
  under its Apache-2.0 option): Pod, Deployment, ReplicaSet, StatefulSet, DaemonSet, Job, CronJob,
  Service, Ingress, Endpoints, NetworkPolicy, ConfigMap, Secret, PersistentVolume(Claim),
  StorageClass, Namespace, Node, ServiceAccount, Role(Binding), ClusterRole(Binding), HPA, CRD and
  the rest the set covers.
- **Fallbacks in the same style** for what the set doesn't cover: a container icon, a generic
  custom-resource icon for CRD instances, and a generic kind icon.
- **Shown where kinds appear**: panel title bars and tabs, the Resource panel's kind list, cards
  in the pod Configuration tab and object viewer, and resource links.
- **Credit**: the About window credits the Kubernetes icon set, and its licence ships with the app.

## Capabilities

### New Capabilities

- `resource-icons`: which icon represents each kind, the fallbacks, where icons appear, and
  attribution.

### Modified Capabilities

(none)

## Impact

- `App/app/assets/icons/kubernetes/` (SVGs + licence), an icon lookup by kind, and the panel title,
  tab, kind-list, card and link renderers.
- The About window's credits.
