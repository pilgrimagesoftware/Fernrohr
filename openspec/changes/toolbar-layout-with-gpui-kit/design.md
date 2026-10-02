# Design

## Context

See proposal.md. The context bar (`ui/context_bar.rs`) and the status bar both rendered one entry per
window context, with the same health; the top bar was the OS title area.

## Decisions

### D1: The status bar owns the contexts
`StatusBarView` absorbs the context bar: each capsule carries name, tunnel, state icon and text, and
elapsed time, problem contexts first. Clicking a capsule makes it the active context; its menu holds
Disconnect with the existing confirmation. The add control follows the capsules and the row scrolls
when it overflows. `context.add` and `context.disconnect` are registered commands, giving both a
keyboard route for the first time. *Alternative:* contexts in the toolbar (this change's first
draft) - rejected by the user as duplicating the status bar.

### D2: Theme switching through one setter
`ui::theme::set` applies the theme to every window and writes only `theme` in `ui.toml`. The status
bar switcher and the `theme.system`/`light`/`dark` commands both call it. With System, windows follow
OS appearance changes for the rest of the session.

### D3: gpui-kit TitleBar + Toolbar
Windows open with a transparent titlebar and `app_owns_titlebar_drag`; a gpui-kit `TitleBar` keeps
drag, double-click zoom and the traffic lights, and holds a `Toolbar` with a 64px icon and the app
name. A dedicated small icon asset keeps rendering and test runs fast.

## Risks / Trade-offs

- [OS appearance following can't be exercised by the test platform] -> covered by the manual check.
- [Enter in the disconnect confirmation dismisses rather than confirms] -> the dialog's safe default;
  keyboard confirmation is Tab to the button and Space.
