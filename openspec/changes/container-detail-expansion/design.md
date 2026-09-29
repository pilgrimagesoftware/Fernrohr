# Design

## Env values: names only, values only when safe

`Container.env` (`Vec<EnvVar>`) carries either a literal `value` or a `value_from` reference to a
ConfigMap/Secret/field. A literal value is safe to show as-is. A Secret-sourced one is not - this
panel already has read access to the Pod object, not blanket read access to arbitrary Secrets, and
even if it did, printing decoded secret values in a detail view by default is the wrong default
for a cluster tool. Render: literal values verbatim, `value_from` entries as their *reference*
("from Secret db-creds key password"), never resolved. ConfigMap-sourced values are lower-stakes
but for consistency get the same treatment - "from ConfigMap app-config key log_level" rather than
a second special case.

## Expansion state and data cost

`ContainerSummary` already holds what the collapsed card needs; the expanded fields (env, volume
mounts, probes, command/args, security context) come straight off the same `Container`/
`ContainerStatus` objects `summarize_containers` already has in hand when it builds the summary -
there's no second fetch, just more fields read from data already present. So the honest choice is
computing the full detail unconditionally in `summarize_containers` (a `ContainerDetail` struct
nested in or alongside `ContainerSummary`) rather than a separate on-demand path - the proposal's
"or a separate ContainerDetail fetched only on expansion" framing turns out to not apply once you
check what data is actually available already; noting that here since the proposal was written
before checking.

Expand/collapse is per-container UI state on `PodDetailPanel`, the same `open_sections`-style
label-keyed set the Managed Fields/Tolerations/Volumes disclosures already use - a fourth kind of
collapsible section, not a new state-tracking mechanism.

## Probes: summarized, not the raw `Probe` union rendered field-by-field

A `Probe` is itself a union (HTTP GET / TCP socket / exec / gRPC) with several timing fields.
Rendering it as one readable line - `"HTTP GET /healthz:8080 every 10s"` - matches how the rest of
this panel treats unions it displays but doesn't need to branch application logic on (see
`format_container_state` for the same pattern already established for `ContainerState`).
