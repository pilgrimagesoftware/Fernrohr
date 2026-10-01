# Spec Delta

## ADDED Requirements

### Requirement: The Window menu lists every open window by its own title

The Window menu SHALL list each open application window as its own entry, labelled with that
window's title, so that two windows connected to different clusters can be told apart from the
menu. Selecting an entry SHALL bring that window forward.

#### Scenario: Two windows on different clusters are distinguishable

- **WHEN** two windows are open, one connected to `staging` and one to `production`
- **THEN** the Window menu lists both as separate entries, each naming its own cluster

#### Scenario: Selecting a window's entry brings it forward

- **WHEN** the user selects a window's entry in the Window menu
- **THEN** that window becomes the active window

#### Scenario: Closing a window drops its entry

- **WHEN** the user closes a window
- **THEN** the Window menu no longer lists an entry for it
