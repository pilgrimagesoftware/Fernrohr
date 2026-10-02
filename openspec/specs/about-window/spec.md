# about-window Specification

## Purpose
Gives the application a real About window: the app's identity, which exact build is running, and
a one-click way to get that build identifier into a bug report, instead of a placeholder box.

## Requirements

### Requirement: Single-instance window

Activating the About action SHALL open the About window, or bring an already-open one to the
front, never a second instance.

#### Scenario: About opened while already open

- **WHEN** the About window is already open and the user activates the About action again
- **THEN** the existing About window is brought to the front
- **THEN** no second About window is opened

#### Scenario: About opened after being closed

- **WHEN** the About window was open, has since been closed, and the user activates the About
  action again
- **THEN** a new About window opens

### Requirement: Window shows app identity and build

The About window SHALL show the packaging icon, the application name, the released version
number, and a build identifier distinguishing which commit and date produced the running binary.

#### Scenario: Version and build are both visible

- **WHEN** the About window is open
- **THEN** it shows the application's icon, name, and released version number
- **THEN** it shows a build identifier containing a date and either a commit hash or an explicit
  statement that the commit is unknown

#### Scenario: Build identifier never shows a blank or raw placeholder

- **WHEN** the running binary was built without access to a git repository or without `git`
  installed
- **THEN** the build identifier states the commit is unknown rather than showing an empty field
  or an internal placeholder token

### Requirement: Build details are copyable

Clicking the version/build text SHALL copy a single line containing the application name,
version, and build identifier to the system clipboard, suitable for pasting into a bug report.

#### Scenario: Copying build details

- **WHEN** the user clicks the version/build text in the About window
- **THEN** the system clipboard contains one line with the application name, version, and build
  identifier

### Requirement: Window is keyboard-dismissible

The About window SHALL close when the user presses Escape while it has focus. On platforms
without a window-chrome convention for closing a window, it SHALL also show a Close control.

#### Scenario: Escape closes the window

- **WHEN** the About window has focus and the user presses Escape
- **THEN** the About window closes

#### Scenario: Explicit close control off macOS

- **WHEN** the About window is open on a platform other than macOS
- **THEN** the window shows a Close button that closes it when activated
