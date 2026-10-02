# Design

## Context

The App's main window currently renders a top bar outside the gpui-kit Toolbar component. The gpui-kit component library (Longbridge) provides a Toolbar component that can standardize layout and styling. The change restructures the top bar to use gpui-kit Toolbar with a specific layout (icon, app name, connected contexts..., add context button) while preserving existing functionality.

See proposal.md for motivation; requirements are in the app-shell spec delta.

## Goals / Non-Goals

**Goals:**
- Replace the current top bar implementation with gpui-kit Toolbar.
- Arrange toolbar contents in the specified order: icon, app name, connected contexts list, add context button.
- Preserve all existing behavior for connected contexts display and the add context action.
- Maintain keyboard-first accessibility and existing styling where appropriate.

**Non-Goals:**
- Changing how contexts are managed or connected.
- Altering the behavior of the add context action beyond integrating it into the new toolbar structure.
- Introducing new toolbar items beyond those specified.

## Decisions

1. **Component adoption**: Use `gpui_component::toolbar::Toolbar` (or the appropriate gpui-kit Toolbar API as used in the codebase) for the main window top bar. Identify the current top bar implementation and replace its container with Toolbar while preserving internal content structure.

2. **Layout composition**: Compose the Toolbar content in order: icon element, app name label, the connected contexts list/view, and the add context button. Use Toolbar's item/section APIs as appropriate to the gpui-kit version used.

3. **Preserve behavior**: Keep the existing connected contexts rendering logic intact; only relocate it inside Toolbar. Keep the add context action/handler intact and ensure the button remains keyboard-focusable and activatable.

4. **Styling integration**: Leverage gpui-kit Toolbar's styling to maintain visual consistency with the component library. Avoid custom styles that conflict with the library's design system.

## Risks / Trade-offs

- [Risk] Toolbar API differences from current top bar layout may require minor adjustments to spacing/alignment → Mitigation: Follow gpui-kit patterns used elsewhere in the codebase.
- [Risk] Regression in layout on different window sizes → Mitigation: Use Toolbar's responsive behavior and existing layout constraints.
- [Risk] Keyboard navigation could be affected if focus order changes → Mitigation: Preserve logical tab order (icon/app name not focusable, then contexts/add context as before).