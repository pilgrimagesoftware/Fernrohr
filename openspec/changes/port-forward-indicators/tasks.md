# Tasks

## 1. Lookup

- [ ] 1.1 Make `PortForwards` observable, with a per-object index and `for_object` / `stop(id)`.
      Verify with unit tests that starting and stopping a forward updates the index and notifies
      observers, and that a stop from Manage Tunnels updates every observer.

## 2. Lists

- [ ] 2.1 Add the Forwards column (icon, count and tooltip) to the Pods and Services lists. Verify
      with view tests that the indicator appears after `shift-f`, disappears after a stop, and
      that the tooltip lists the local address and target port.
- [ ] 2.2 Remove the list-panel success notice, and route start failures to a notification.
      Verify with a test that a pod with no ports produces a notification and no panel notice.

## 3. Pod detail panel

- [ ] 3.1 Add the forward strip above the tabs, with copy and stop icon buttons and tooltips.
      Verify with view tests that it is shown while forwards exist, that stop and copy work, and
      that it is hidden when there are none.
- [ ] 3.2 Add start, or address plus copy and stop, icon buttons next to each container port in
      the Containers tab. Verify with mouse tests that clicking start forwards that port without a
      prompt, and that stop releases it.

## 4. Commands

- [ ] 4.1 Register `PortForwardPod` in the pod detail context. Add `StopPortForward` (picker when
      there are several) to the Pods, Services and pod detail contexts, all with `!Input`, and add
      them to the hint rows. Verify with keystroke tests, and with the prefix-aware conflict check
      against every default.

## 5. Gates

- [ ] 5.1 `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` all
      pass, and touched files stay under 500 lines.

## 6. Verification

- [ ] 6.1 Manual check, needing user confirmation:
      - Forward a pod with `shift-f` and see the row indicator.
      - Open its detail panel and see the strip. Copy the address and `curl` it.
      - In the Containers tab, click a second port's forward icon.
      - Stop one forward from the strip and the other with Stop Port Forward.
      - Confirm all the indicators clear.
