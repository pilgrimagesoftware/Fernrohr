# Design

## Context

The App submodule contains the Rust/GPUI UI. Resource detail panes render selected resources in different forms (summary fields, YAML view, etc.). The YAML view currently lacks scrolling and folding for large manifests. Status rendering is plain text without color cues. Managed fields are shown as raw structures. Copy affordances are missing for both the resource name and individual field values. The init container waiting state is not clearly represented.

See proposal.md for motivation. Requirements are defined in the spec delta for `resource-details`.

## Goals / Non-Goals

**Goals:**
- Fix init container waiting state rendering so it's clearly shown.
- Add vertical scrolling and folding/collapsing to the YAML view in detail panes.
- Add "Copy Resource Name" action with keybinding/menu integration (keyboard-first).
- Add hover-activated copy buttons for specific fields (pod container image, ConfigMap key/value, similar cases) and ensure they work with both mouse and keyboard navigation.
- Colorize status values (starting with Pod status) for quick visual scanning.
- Replace raw managed fields with a collapsible disclosure control (consistent with ConfigMap presentation).

**Non-Goals:**
- Changing the Kubernetes API schema or data models.
- Adding new resource types or altering discovery.
- Implementing clipboard history or bulk copy; simple clipboard copy only.

## Decisions

1. **Init container waiting state**: Inspect the code that renders Pod status/init containers in detail views. Ensure waiting init containers render with explicit "Waiting" state (including reason/message) rather than being conflated with completed/terminated states. Follow existing status formatting patterns.

2. **YAML view enhancements**: Extend the YAML renderer/view used in detail panes to support:
- Vertical scroll (e.g., wrap in a scrollable container consistent with gpui-component/Table/Dock patterns)
- Collapsible/foldable nodes for nested YAML (hierarchical folding). Leverage or extend existing YAML rendering utilities; avoid introducing heavy new dependencies.

3. **Copy actions (resource name)**: Register a new command/action `copy_resource_name` in the command registry with appropriate title, default keybinding (keyboard-first), and context predicate (resource selected in detail pane/resource list). Wire it to the detail pane and ensure it appears in menus where relevant. Copy to system clipboard using GPUI's clipboard APIs.

4. **Per-field copy affordances**: For targeted fields (container image, ConfigMap key, ConfigMap value), render a small copy button/icon that appears on hover of the field row/value. Ensure the button is keyboard-focusable and can be activated via keyboard (Enter/Space) to satisfy keyboard-first requirement. Generalize the pattern if other fields benefit, but scope to the named cases.

5. **Status colorization**: Add semantic styling classes/colors for status values based on state (e.g., Running/Succeeded → positive, Pending/Waiting → neutral/warning, Failed/Error/Terminated → error). Start with Pod status; design for reuse across other resources later. Follow existing theme/styling conventions in the codebase.

6. **Managed fields disclosure**: Replace flat rendering with an expandable/collapsible disclosure control (tree-like or section-based), mirroring how ConfigMap key/value pairs are presented. Default state should balance readability (likely collapsed for large managedFields) while remaining discoverable.

## Risks / Trade-offs

- [Risk] YAML folding complexity for large deeply nested manifests → Mitigation: Implement incremental folding with reasonable defaults; keep existing plain view behavior if needed; test with realistic manifests.
- [Risk] Hover-only copy buttons may not be discoverable → Mitigation: Also support keyboard activation and consider making buttons always visible for focusable elements per keyboard-first guidelines.
- [Risk] Color-only status cues may not be accessible → Mitigation: Include text labels and ensure sufficient contrast; follow accessibility conventions in GPUI theming.