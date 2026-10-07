# Spec Delta

## ADDED Requirements

### Requirement: Each pod's logs open in a panel of their own
The application SHALL by default open a pod's logs in a Logs panel for that pod alone, so the
user can keep several pods' logs open side by side, rather than retargeting a single Logs panel to
whichever pod was opened last. Opening the same pod's logs again SHALL focus its panel rather than
add a second, and opening another container of that pod SHALL switch its panel to that container.
A pod's panel SHALL stay on its pod when other pods are selected later, and SHALL restore as that
pod's panel from a saved layout.

#### Scenario: Per-pod panels are the default
- **WHEN** a user opens one pod's logs and then another pod's, with the default preference
- **THEN** each pod's logs are in a panel of their own, and the first panel is unaffected

#### Scenario: Reopening a pod's logs focuses its panel
- **WHEN** a user opens the logs of a pod whose logs panel is already open
- **THEN** that panel is focused, not duplicated

#### Scenario: Another container of an open pod
- **WHEN** a user opens a different container's logs of a pod whose logs panel is open
- **THEN** that pod's panel switches to the container, and no second panel opens

#### Scenario: A pod's panel ignores later selections
- **WHEN** a user selects another pod without opening its logs
- **THEN** each pod's logs panel keeps showing its own pod

### Requirement: Reuse is a preference, and every open can flip it once
The application SHALL let the user choose, in Settings and from the command palette, whether
opening a pod's logs opens its own panel (the default) or reuses one shared Logs panel that follows
the selection. Every way of opening logs - `l` in the Pods list and a pod's detail panel, the Pods
row's menu, and the palette - SHALL have a flipped twin (`shift-l`, a menu item, a palette
command) that opens them the other way from the preference, for that one open.

#### Scenario: The flip opens the shared panel by default
- **WHEN** a user presses `shift-l` on a pod with the default preference
- **THEN** the pod's logs open in the one shared Logs panel, retargeting it from any pod it showed

#### Scenario: Reuse preference
- **WHEN** the preference is set to reuse one panel and the user opens two pods' logs with `l`
- **THEN** the one shared Logs panel shows the second pod's logs, and `shift-l` opens a pod's
  logs in a panel of their own

#### Scenario: The preference is kept
- **WHEN** the user changes the preference
- **THEN** it applies at once and is saved, so it holds after a restart
