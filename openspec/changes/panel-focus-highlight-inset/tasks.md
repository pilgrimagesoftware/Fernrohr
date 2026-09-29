# Tasks

## 1. Inset the focus border

- [x] 1.1 Change `focus_frame` to draw its border against an inset inner element rather than the
  full-size outer one, so the border never sits flush with a panel's outer edge. Verify: existing
  `focus_border`/`a_focused_panel_uses_the_primary_border` tests still pass (the color logic is
  unchanged; only the geometry wrapping it changes).
- [ ] 1.2 Manual check: focus a panel occupying the window's bottom edge and confirm the border
  is no longer visibly clipped by the window's rounded corner. **Needs user confirmation against
  a running build.**

## 2. Full verification

- [x] 2.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass (185/185, excluding the pre-existing unrelated `placeholder_remembers_the_kind_it_was
  _opened_for` flake).
- [ ] 2.2 Manual smoke test: cycle focus between two panels, including one at the window's bottom
  edge, and confirm the border reads cleanly in both cases. **Needs user confirmation.**
