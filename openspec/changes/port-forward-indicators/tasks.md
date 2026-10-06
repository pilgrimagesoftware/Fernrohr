# Tasks

## 1. Lookup

- [x] 1.1 Make `PortForwards` observable, with a per-object index and `for_object` / `stop(id)`.
      Verify with unit tests that starting and stopping a forward updates the index and notifies
      observers, and that a stop from Manage Tunnels updates every observer.

## 2. Lists

- [x] 2.1 Add the Forwards column (icon, count and tooltip) to the Pods and Services lists. Verify
      with view tests that the indicator appears after `shift-f`, disappears after a stop, and
      that the tooltip lists the local address and target port.
- [x] 2.2 Remove the list-panel success notice, and route start failures to a notification.
      Verify with a test that a pod with no ports produces a notification and no panel notice.

## 3. Pod detail panel

- [x] 3.1 Add the forward strip above the tabs, with copy and stop icon buttons and tooltips.
      Verify with view tests that it is shown while forwards exist, that stop and copy work, and
      that it is hidden when there are none.
- [x] 3.2 Add start, or address plus copy and stop, icon buttons next to each container port in
      the Containers tab. Verify with mouse tests that clicking start forwards that port without a
      prompt, and that stop releases it.

## 4. Commands

- [x] 4.1 Register `PortForwardPod` in the pod detail context. Add `StopPortForward` (picker when
      there are several) to the Pods, Services and pod detail contexts, all with `!Input`, and add
      them to the hint rows. Verify with keystroke tests, and with the prefix-aware conflict check
      against every default.

## 5. Manage Tunnels and stop confirmation

- [x] 5.1 Put a horizontal divider between the Tunnels section and the Port forwards section of the
      Manage Tunnels window, using the theme's border color and the existing spacing tokens and
      matching the app's other section separators. Verify with a view test that the divider
      renders between the two sections, and still renders when the Port forwards section is
      empty.
- [x] 5.2 Make every stop path confirm first: the strip, the container port, Stop Port Forward and
      Manage Tunnels' Stop. Use one shared dialog that names the pod or service, local address and
      port with the names set apart from the text, and reuse #143's confirmation styling. Verify
      with keystroke and mouse tests on each path that Escape or Cancel keeps the forward and Enter
      or Confirm stops it.

## 6. Gates

- [x] 6.1 `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` all
      pass, and touched files stay under 500 lines.

## 7. Verification

- [ ] 7.1 Manual check, needing user confirmation:
      - Forward a pod with `shift-f` and see the row indicator.
      - Open its detail panel and see the strip. Copy the address and `curl` it.
      - In the Containers tab, click a second port's forward icon.
      - Stop one forward from the strip and the other with Stop Port Forward, cancelling the
        confirmation once before confirming.
      - Confirm all the indicators clear.
      - Open Manage Tunnels and check the divider between Tunnels and Port forwards.

## Notes

- Implemented in pilgrimagesoftware/Fernrohr-App#137, rebased onto develop after #143. There is
  one commit per section, plus a refactor moving the stop confirmation onto #143's
  `ui::confirm_dialog`.
- A Service forward reaches one Pod behind the Service, so its request alone can't say which
  Service it came from. Each forward records the objects it was started from. A Pod's forwards are
  those reaching it, and a Service's are those started from it.
- `ForwardSummary` has no container: a request carries only the pod and port. Section 3's container
  ports will match on the port.
- Start failures use gpui-kit's notification. gpui-component keeps a window's notification list
  private, so tests read a `cfg(test)` record that `notify_failure` keeps.
- Every stop path asks through `ui::forward_stop`, which uses #143's shared confirmation: the
  strip, a container port, Stop Port Forward and Manage Tunnels' Stop. Stop Port Forward
  (`ctrl-shift-f`, `… && !Input` in the Pods list, core Services lists and pod detail) confirms
  directly for one forward. With several it asks which first, in a picker whose Cancel shows its
  key.
- #143's shared `pods::forward_pod` now reports start failures itself as a notification. Success
  needs no report, so the pod detail panel's success notice is gone too: its strip shows the
  forward.
- Following the new icon-buttons rule, Manage Tunnels' Stop is now an icon button with a tooltip.
  `ui::icon_tooltip` gives that tooltip a debug selector, so tests can assert it.
- The Services list's Stop Port Forward has no test of its own. It shares
  `forward_stop::stop_one_of` with the Pods list and pod detail, which are tested by keystroke.
