# Tasks

## 1. Context default state

- [x] 1.1 Add an optional per-context default namespace field to persisted workspace/session state and verify existing state files still deserialize.
  App#110: `WorkspaceConfig::namespace_defaults` (context to namespace list, serde-defaulted). Verified
  by `config::workspace` tests (an older file loads with none; round trip) and
  `util::shell::warp::tests::the_default_is_saved_and_restored_with_the_workspace`.
- [x] 1.2 Apply the context default when constructing a new namespaced panel and verify a panel opened after setting the default starts in that namespace.
  `open_target_in` scopes a namespaced kind list opened with no scope to the default. Verified by
  `warp::tests::warp_all_moves_the_context_and_sets_its_default` (a list opened afterwards) and
  `with_no_default_a_new_list_opens_on_all_namespaces`.

## 2. Commands and propagation

- [x] 2.1 Register the "Warp all to namespace" action with the existing command system and verify it appears in the palette for a Pods panel.
  `pods.warp_all_namespace`, `shift-w`, Pods-scoped, palette only. Verified by
  `warp::tests::warp_all_is_a_pods_scoped_palette_command`.
- [x] 2.2 Update all open namespaced panels in the active context and set the context default, leaving cluster-scoped panels unchanged; verify with focused-panel and multi-panel tests.
  `MainWindow::warp_context`, the reusable helper. Verified by
  `warp::tests::warp_all_moves_the_context_and_sets_its_default` (namespaced lists move, cluster-scoped
  and other contexts stay) and `the_focused_warp_stays_local`.

## 3. Verification

- [x] 3.1 Run the relevant Rust unit and integration tests and verify existing single-panel warp behavior is unchanged.
  893 passed, 2 ignored; `w` unchanged (`the_focused_warp_stays_local`).
