# Tasks

## 1. Open path

- [ ] 1.1 Add `OpenMode { Foreground, Background }` to `open_target_in`, with existing callers
      passing `Foreground`. Background inserts the tab inactive (restoring the group's previously
      active tab if the dock activates it) and skips focus, and returns early for an already-open
      panel. Verify with shell tests that:
      - a background open adds an inactive tab and leaves focus on the list
      - the list stays its group's active tab when the new tab joins that group
      - an already-open object changes nothing
      - foreground opens are unchanged

## 2. List rows

- [ ] 2.1 Register `OpenInBackground` (`secondary-enter`, `!Input`) for the Pods panel and
      `ObjectListPanel`, and show it in both hint rows. Verify with keystroke tests that
      `cmd-enter` opens in the background with the selection unchanged, does nothing inside the
      filter field, and doesn't conflict with any registered default.
- [ ] 2.2 Handle a modifier-click and a middle-click on rows in both tables. Verify with mouse
      tests that the clicked row's object opens in the background and the selection stays put.

## 3. Links

- [ ] 3.1 Make links honor a modifier-click and a middle-click. Verify with a test that a pod's
      owner link opens the owner in the background with focus kept in the pod detail panel.

## 4. Gates

- [ ] 4.1 `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` all
      pass, and touched files stay under 500 lines.

## 5. Verification

- [ ] 5.1 Manual check: in a Pods panel, `cmd`-click three pods and `cmd-enter` a fourth. Four
      detail tabs appear without focus leaving the list, and the selection doesn't move.
      Middle-click a Deployment row. `cmd`-click an owner link in a pod detail panel.
      **Needs user confirmation.**
