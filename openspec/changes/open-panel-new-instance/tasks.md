# Tasks

## 1. Instances in scope and key

- [x] 1.1 Add `instance: u32` to `PanelScope` (D1) and copy it into `PanelKey`. Add the title and
  tab suffix ` (n)` for an instance of 1 or more, shown as `instance + 1`. Verify with unit tests in
  `ui/panel/title/tests.rs`: instance 0 shows no suffix, instance 1 shows ` (2)`, and keys differ by
  instance.
- [x] 1.2 Teach `open_target_in` the three ways to match (D2): any instance by default, a new
  instance with the lowest free number for `OpenMode::NewPanel` (D3), and exactly one instance for
  layout Add. Verify with shell tests: a default open focuses the first of two instances; NewPanel
  numbers 2, then 3, then reuses a freed 2.

## 2. Open in New Panel for the focused panel

- [x] 2.1 Register `panel.open_new_instance` ("Open in New Panel", `cmd-shift-d`, View menu) and
  have `MainWindow` open the focused panel's key again with `NewPanel` in its tab group, or, for a
  typed label panel, a new empty typed panel (D4, D6). Verify with shell tests using real
  keystrokes: `cmd-shift-d` on a Deployments list, a pod detail, an object detail and the Events
  browser each opens a second one with focus; the first instance keeps its own namespace scope when
  the second is re-scoped.

## 3. Row routes in lists

- [x] 3.1 Add `object_list.open_new_panel` and `pods.open_new_panel` (`shift-enter`, `!Input`), with
  `OpenPodInNewPanel` for the Pods list (D3, D4). Verify with keyboard tests: `shift-enter` on a row
  whose panel is open opens a second; with the filter field focused it doesn't.
- [x] 3.2 Add `alt`-click to `ui::background_rows::row`, opening the clicked row in a new panel and
  leaving the selection as a `cmd`-click does. Add Open in New Panel to the object list's and the
  Pods table's row context menus. Verify with mouse tests: an `alt`-click and the context-menu item
  each open a second panel on a row that's already open.

## 4. Saved layouts

- [x] 4.1 Save and restore `instance` in every panel kind (Pods, object list, events browser,
  placeholder, Logs, pod detail, object detail, exec) through a shared helper, and read it in
  `panels::panel_key` (D5). Verify with tests: a dock dump with two instances of a Deployments
  list, and two of a pod detail, restores both with their numbers and scopes; a state without
  `instance` restores as 0.
- [x] 4.2 Make layout Add match instances exactly (D2). Verify with a load test: with instance 1 of
  a Service detail open, adding a layout holding instances 1 and 2 opens only instance 2.

## 5. Documentation

- [x] 5.1 Document Open in New Panel and numbered titles in `docs/site/latest/windows-panels.md`,
  the row routes in `docs/site/latest/resources.md`, and the new commands in
  `docs/site/latest/reference/keybindings.md`, with both routes and placeholder names. Verify: the
  docs-site check (`bundle exec ruby _scripts/check.rb`) and `cargo test` pass.

Implementation notes (Fernrohr-App):

- The instance is `PanelScope.instance: u32`, with `PanelKey.instance` copied from it. Titles are
  numbered centrally in `panel_title::title_element` via `numbered`, which every panel kind's title
  goes through. Saving goes through `panel_title::with_instance` and `saved_instance`.
- `open_target_in` delegates to `open_target_as(.., Instance::Any)`; layout Add calls
  `open_target_as(.., Instance::Exactly(n))`.
- Split Group (`arrange::split_group`) already built a second panel on the focused panel's key,
  bypassing dedup. Its copy now takes the next free instance number, so it is keyed and titled
  apart like any other instance.
- Tests: `util/shell/new_instance_tests.rs` (2.1 with `cmd-shift-d` keystrokes, 1.2, 4.1, 4.2),
  `object_list/panel/tests/background.rs` and `pods/background/tests.rs` (3.1, 3.2),
  `ui/panel/title/tests.rs` (1.1).

## 6. Integration

- [x] 6.1 Run `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test`
  in `App/`. Verify: all pass.
- [ ] 6.2 Check manually: duplicate a Deployments list with `cmd-shift-d` and scope the two
  differently; `alt`-click a Service row twice; `shift-enter` a pod; save the layout, reload it with
  Replace and with Add, and check that every instance comes back numbered with its own scope.
  Record no cluster-identifying details.
