# Proposal

## Why

Resource discovery already sorts kinds by API group then kind name with core first, but the
resource picker presents them as one flat list. As clusters accumulate CRDs (cert-manager,
Istio, Argo, vendor operators), the flat list grows long and mixes unrelated APIs together,
making it hard to find a kind by scanning. Grouping by API group turns an alphabetic scan into a
recognizable landmark search.

## What Changes

- Present the resource kind picker as sections headed by API group (`core` first, then other
  groups alphabetically), with kinds listed under their group in the existing sort order.
- Collapse/expand a group's section, remembered per cluster connection for the session.
- Keep the picker's existing fuzzy filter working across all groups at once; a matching filter
  temporarily expands any collapsed groups containing a match and shows only groups with at
  least one match.
- Keyboard navigation (arrow keys, type-ahead) moves between kinds within and across groups in
  the same order the groups are drawn, and a keybinding toggles the focused group's
  collapsed state.

## Capabilities

### New Capabilities

- `resource-group-navigation`: the resource kind picker's grouping of discovered kinds by API
  group, including collapse/expand state and how filtering and keyboard navigation interact
  with groups.

### Modified Capabilities

(none)

## Impact

- `App/src/ui/picker.rs` (or wherever the kind/resource picker list is rendered) and its
  `DiscoveredKind` consumption.
- `App/src/cluster/discovery.rs`'s existing group-then-kind sort (reused, not changed).
- Per-cluster-connection session state (new: collapsed-group set).
