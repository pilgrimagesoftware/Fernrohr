# Proposal

## Why

The main window's top bar currently sits outside the gpui-kit Toolbar component. Moving it into a gpui-kit Toolbar and restructuring the layout (icon, app name, connected contexts list, and an "add context" button) improves visual consistency, leverages the component library's styling/behavior, and makes the toolbar more maintainable and extensible.

## What Changes

- **Refactor**: Use gpui-kit Toolbar for the main window top bar.
- **Layout restructure**: Move the existing top bar content into the Toolbar with the following layout: icon, app name, connected contexts..., and an add context button.
- **Maintain functionality**: Preserve existing behavior for connected contexts display and adding contexts while adapting to Toolbar's API.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `app-shell`: Updates the main window chrome/layout to use gpui-kit Toolbar for the top bar with the specified layout elements (icon, app name, connected contexts, add context button).

## Impact

- Affects the App submodule's main window/top bar UI (likely in the app shell/window chrome components).
- Leverages gpui-kit components (Toolbar). No external dependency changes expected if already present.
- Preserves keyboard-first behavior and existing actions for adding contexts.