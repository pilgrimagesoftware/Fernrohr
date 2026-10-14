# Tasks

## 1. Switching context in `MainWindow`

- [x] 1.1 Add the panel-to-window switch request (D1) and its `MainWindow` handler. The handler
  opens a typed panel in the requested context with the requesting panel's selector and container
  filter, scoped to all namespaces, in the requesting panel's dock slot, then closes the requesting
  panel. Verify with a shell test in `util/shell/label_logs_tests/`: two contexts over two fake
  clusters; the switch leaves one typed panel, keyed to the new context, in the old tab position.
  It streams the new cluster's matching pod in a non-default namespace, and no stream from the old
  cluster remains.
- [x] 1.2 When the requested context already has a typed panel (D2), focus that panel and leave the
  requesting one unchanged. Verify with a shell test: two typed panels, a switch from one to the
  other's context focuses the other, and both keep their contexts and streams.
- [x] 1.3 Check that a switched panel saves its new context and that the missing-context path
  covers it (D5). Verify with a saved-layout test: switch, save, load into a window holding both
  contexts, and the panel searches the new one. Load into a window holding only the old context,
  and a placeholder names the new one.

## 2. The context control and its command

- [x] 2.1 Give the typed panel the window's context list (from `sync_context_children`, as the
  Resource panel gets it), and draw the dropdown in its bar beside the namespace picker only when
  there is more than one (D3). Verify with a panel test: no control with one context, the control
  with the current context checked with two, and it disappears when the second context is closed.
- [x] 2.2 Picking a context in the dropdown emits the request from 1.1. Verify with a mouse test:
  clicking the control and an item switches the panel, as in 1.1.
- [x] 2.3 Register "Logs: Pick Context…" (`logs.pick_context`, default `x`, Logs panel
  `KeyContext`) to open the dropdown, and show its key in the hint row from the live keymap while
  the control is visible (D4). Verify with a keyboard test (`simulate_keystrokes`): `x`, `down`,
  `enter` switches the panel; `escape` closes the dropdown without switching. Also check that the
  command is in the palette and that a `keymap.toml` override rebinds it.
- [x] 2.4 Document the control: add an Open Logs by Label section to
  `docs/site/latest/logs-shells.md` covering the selector, namespace scope and context control,
  with both routes and placeholder context names (`my-cluster`). Add the command and key to
  `docs/site/latest/reference/keybindings.md`. Verify: the docs-site check
  (`bundle exec ruby _scripts/check.rb`) passes and `cargo test` passes.

Implementation notes (Fernrohr-App, `util/logs/labels/context.rs`, `util/shell/logs_context.rs`):

- The request is an action, `SwitchLabelLogsContext { panel, context }`, not a panel event as D1
  sketched. Panels restored with a layout are never subscribed by `MainWindow`, and an action
  dispatched from the panel reaches the window either way.
- The panel reads its window's contexts at render (`util::shell::window_contexts`, beside
  `window_context_count`) rather than having them pushed through `sync_context_children` (D3).
- The keyboard route (D4) opens a "Pick Context" dialog list in the `links::go_to` style, with
  filterable rows, arrows, Enter and Escape, rather than the button's popup menu, which can't be
  opened from the keyboard. The mouse uses the popup menu. The control's `Kbd` shows the live key.
- 1.3's test round-trips the dock dump through `DockArea::load` in a window holding both contexts.
  The missing-context half is the generic placeholder path already covered by
  `saved_layouts/load/tests/missing_context.rs`, which applies to every saved panel unchanged.

## 3. Integration

- [x] 3.1 Run `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test`
  in `App/`. Verify: all pass.
- [ ] 3.2 Check manually in a window holding two contexts: open Logs by Label from the first
  context's Pods list, switch to the second by keyboard and by mouse, and check that its pods
  stream. Record no cluster-identifying details.

## Workflow follow-up

- Archive this change only after `selector-logs-panel` is archived, since its delta adds to the
  `selector-logs` capability that change introduces.
