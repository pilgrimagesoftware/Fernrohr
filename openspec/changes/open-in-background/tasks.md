# Tasks

## 1. Open path

- [x] 1.1 Add `OpenMode { Foreground, Background }` to `open_target_in`, with existing callers
      passing `Foreground`. Background inserts the tab inactive (restoring the group's previously
      active tab if the dock activates it) and skips focus, and returns early for an already-open
      panel. Verify with shell tests that:
      - a background open adds an inactive tab and leaves focus on the list
      - the list stays its group's active tab when the new tab joins that group
      - an already-open object changes nothing
      - foreground opens are unchanged

## 2. List rows

- [x] 2.1 Register `OpenInBackground` (`secondary-enter`, `!Input`) for the Pods panel and
      `ObjectListPanel`, and show it in both hint rows. Verify with keystroke tests that
      `cmd-enter` opens in the background with the selection unchanged, does nothing inside the
      filter field, and doesn't conflict with any registered default.
- [x] 2.2 Handle a modifier-click and a middle-click on rows in both tables. Verify with mouse
      tests that the clicked row's object opens in the background and the selection stays put.

## 3. Links

- [x] 3.1 Make links honor a modifier-click and a middle-click. Verify with a test that a pod's
      owner link opens the owner in the background with focus kept in the pod detail panel.

## 4. Gates

- [x] 4.1 `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` all
      pass, and touched files stay under 500 lines.

## 5. Verification

- [ ] 5.1 Manual check: in a Pods panel, `cmd`-click three pods and `cmd-enter` a fourth. Four
      detail tabs appear without focus leaving the list, and the selection doesn't move.
      Middle-click a Deployment row. `cmd`-click an owner link in a pod detail panel.
      **Needs user confirmation.**

## Notes

- Implemented in pilgrimagesoftware/Fernrohr-App#135, one commit per section 1-3.
- The dock always shows a newly added tab, so a background open records which tab each centre
  group shows and shows those again in the same dock update. A test confirms the restore is
  needed.
- GPUI runs every click listener on a row, so a modified click can't stop the table from
  selecting the clicked row. `ui::background_rows` notes the modified click on the delegate, and
  the panel's `SelectRow` handler puts the old selection back before anything follows the clicked
  row. That is what keeps `SelectedPod`, and so Logs, from moving. A modified double-click's
  `DoubleClickedRow` is swallowed so it doesn't also open in the foreground. A middle-click is an
  aux click, which the table ignores.
- Pods rows open through a new data action, `nav::OpenPodInBackground`, rather than `ShowPodDetail`,
  which reads `SelectedPod`. `OpenListedObject` and `FollowReference` carry an `OpenMode`.
- The keyboard default is written `secondary-enter`, as the design says. It parses to `cmd-enter`
  on macOS.
