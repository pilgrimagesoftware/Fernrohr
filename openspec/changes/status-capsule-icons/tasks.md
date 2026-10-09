# Tasks

## 1. Capsule layout

- [ ] 1.1 Reorder the capsule body to context name, `[tunnel]`, state icon, and elapsed time (non-connected only), with the tunnel and elapsed time in Manrope, one step smaller, in `muted_foreground`; verify render tests for a bound connected capsule, an unbound capsule, and a reconnecting capsule.
- [ ] 1.2 Remove the inline state text and add a tooltip on the state icon with the state, elapsed time, and reason where applicable; verify tests find the icon by debug selector and assert the tooltip for connected, reconnecting, and failed states.
- [ ] 1.3 Assert in a unit test that every `ContextHealth` state maps to a distinct icon.
