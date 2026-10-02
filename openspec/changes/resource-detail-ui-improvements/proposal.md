# Proposal

## Why

The resource detail panes have usability gaps: init containers in waiting state aren't clearly represented (causing confusion when debugging pod startup), the YAML view is not scrollable and lacks folding for large manifests, common values (like container images, ConfigMap keys/values) can't be copied quickly, status values lack visual cues, and managed fields are shown in a raw/verbose form that’s hard to scan. These small UX fixes improve day-to-day navigation and debugging speed.

## What Changes

- **Bugfix**: Correct init container status representation so waiting init containers are displayed meaningfully (not mislabelled/completed).
- **YAML View**: Enable vertical scrolling for large manifests; add folding/collapsing support for nested YAML structures.
- **Copy UX**: Add a keybinding/menu item to copy the name of the currently selected resource. Add hover-activated copy buttons in the detail pane to copy specific values (e.g. pod container image, ConfigMap key, ConfigMap value, and similar fields where copying is useful).
- **Status Visualization**: Colorize status values (starting with Pod status, with consideration for other resources) to make states easier to scan.
- **Managed Fields**: Replace the raw managed fields display with a disclosure control (expand/collapse), consistent with how ConfigMap keys/values are presented, to reduce visual noise.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `resource-details`: Extends the resource detail pane UI to support copy actions (resource name, per-field values), improved YAML rendering (scrolling and folding), colored status indicators, and a collapsible managed fields view.

## Impact

- Affects UI components in the App submodule (resource detail pane, YAML viewer, status rendering, managed fields UI)
- No API or schema changes; purely presentation/UX improvements
- Keyboard navigation and command palette must accommodate new actions (copy resource name) in line with keyboard-first conventions