# Design

## The constraint that decides the whole design: categories cannot come from the API group

The obvious implementation is a map from API group to section - `apps` is
workloads, `networking.k8s.io` is network. It does not work, and it fails on the
single most important group there is.

The **core group** (`""`, served at `/api/v1`) has no name to key on, and it
contains kinds belonging to every category:

| Core-group kind | Category |
| --- | --- |
| `Pod`, `ReplicationController` | Workloads |
| `Service`, `Endpoints` | Network |
| `ConfigMap`, `Secret`, `LimitRange`, `ResourceQuota` | Config |
| `PersistentVolume`, `PersistentVolumeClaim` | Storage |
| `ServiceAccount` | Access Control |
| `Namespace`, `Node`, `Event`, `ComponentStatus` | Cluster |
| `PriorityClass`, `RuntimeClass` | Cluster |

So the mapping is keyed on **`(group, plural)`**, with an empty `group` meaning
core. This is not an implementation detail we invented: the Kubernetes API
reference has to enumerate core-group resources *separately inside each of its
own categories* for exactly this reason, which also makes it a good source to
transcribe the built-in rows from.

`DiscoveredKind` already carries `gvk` and `plural`, so nothing new is needed
from discovery to key on this.

## Why a static table rather than asking the cluster

Two sources of category data were considered.

**A static table in the app.** Deterministic, needs no extra API call, no extra
RBAC, and works before any connection has succeeded. This is what OpenLens,
FreeLens and k9s all do.

**A `categories` annotation on the CRD**, which is the k9s convention and would
let an operator place their own CRDs. Rejected *for now*, for two reasons.
Reading it means listing `CustomResourceDefinition` objects, which is a real
permission the user may not have granted - and a permission failure here would
degrade the whole panel. And almost no operator sets it, so the common case
gains nothing while the failure mode is new.

This is a non-goal rather than a rejection on principle. The lookup stays a
single function, so a later task can consult the CRD and fall back to the
table. What it must not do is let a missing annotation or a forbidden list turn
into an error.

## The taxonomy

OpenLens/FreeLens's sections, in the order they present them:

1. **Workloads** - Pod, ReplicationController, Deployment, ReplicaSet,
   DaemonSet, StatefulSet, Job, CronJob, HPA, PDB
2. **Config** - ConfigMap, Secret, ResourceQuota, LimitRange, PriorityClass
3. **Network** - Service, Endpoints, EndpointSlice, Ingress, IngressClass,
   NetworkPolicy, Gateway/HTTPRoute
4. **Storage** - PersistentVolume, PersistentVolumeClaim, StorageClass,
   CSIDriver, CSINode, VolumeAttachment
5. **Cluster** - Namespace, Node, Event, ComponentStatus, RuntimeClass, Lease
6. **Access Control** - ServiceAccount, Role, RoleBinding, ClusterRole,
   ClusterRoleBinding
7. **Custom Resources** - everything unmatched, which is every CRD

**Custom Resources is the fallback, not an afterthought.** Any kind not in the
table lands there, so a CRD-heavy cluster gets one section that grows rather
than kinds sprayed into sections chosen for a different API's conventions.

The order is **fixed, not alphabetical**. Alphabetical would put Access Control
first and Storage last, and Workloads - the section a user opens most - in the
middle. Fixed order means Workloads is always at the top, and a section that is
empty for a given cluster takes no space at all.

## Collapsed state

Per window, held in the `ResourcePanel`, default expanded, **not persisted**.
Same reasoning as 11.3: this is a transient view preference, and the
`ui.toml` file is for things a user sets once and expects to still be true
tomorrow. Persisting it would also mean a user who collapsed everything to get
a full-screen table on one cluster starts the next session with an empty-looking
panel and no obvious way to tell why.

## How filtering interacts with collapsed sections

The one genuinely fiddly decision. A collapsed section whose rows match the
filter has two defensible behaviours: stay collapsed and show a match count, or
expand and show the rows.

**It expands.** A filter is a question - "show me Ingress" - and an answer the
user cannot see is a broken filter. So while a filter is active, any section
with at least one match renders expanded regardless of its collapsed state, and
sections with no matches are hidden entirely rather than shown as an empty
heading.

The stored collapsed state is *not* mutated by this, so clearing the filter
returns the panel to exactly the shape the user left it in. That is the whole
reason the state is kept separately from what is rendered.

## Matching

Case-insensitive substring against the row's label, its kind, its plural, and
its API group. Substring rather than regex or fuzzy: the audience types `ing`
expecting Ingress, and a regex box in a 240px panel is a trap. Matching the
group too means `batch` finds CronJob even though neither word is in its label.
