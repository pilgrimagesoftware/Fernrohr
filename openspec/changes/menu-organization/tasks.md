# Tasks

## 1. Menu organization

- [x] 1.1 Context menu: context actions first, then a separator, then all tunnel actions. Verify with a menu-structure test asserting the order and the separator.
- [x] 1.2 View menu: group appearance, Resource panel, panel layout and table-column items with separators between groups. Verify with a menu-structure test listing the groups in order.
- [x] 1.3 Navigate menu: keep only focus-next/previous panel, Focus Resources and tab commands; drop every panel-scoped command's menu assignment (Pods, Logs, list panels, fit columns), keeping them in the palette and keymap. Verify with a test that Navigate holds exactly the global set and that the dropped commands still resolve in the palette; confirm the menu is not rebuilt on focus change.
- [x] 1.4 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` pass.
- [ ] 1.5 Manual check: open Context, View and Navigate and confirm the grouping.
