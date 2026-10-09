# exec-terminal Specification

## Purpose
Defines the embedded terminal that hosts an interactive exec session into a container: how input
reaches the container, how output and screen state are shown, and how the terminal behaves as the
panel resizes and the session ends.

## Requirements

### Requirement: Shell sessions run in an embedded terminal

A shell session panel SHALL host a terminal emulator connected to an exec session opened with a
TTY. The panel SHALL render the container's screen output, including colors, cursor positioning,
full-screen programs and the alternate screen, the way a standalone terminal would.

#### Scenario: Full-screen program

- **WHEN** the user runs `top` in a shell session panel
- **THEN** `top` draws and refreshes its full-screen display inside the panel, and quitting it restores the prior screen

#### Scenario: Colors

- **WHEN** a command in the session prints ANSI-colored output
- **THEN** the panel shows the colors, mapped to the application theme's terminal palette

### Requirement: Terminal input goes to the container

While a shell session panel has focus and its session is running, the application SHALL send
every key press, including control keys (Ctrl-C, Ctrl-D, Ctrl-Z), Tab, Escape, arrow keys and
function keys, to the container as terminal input. It SHALL NOT trigger application commands from
those keys, except for a small reserved set:

- the platform's copy and paste shortcuts
- the command palette
- the panel focus and tab navigation shortcuts

The reserved set SHALL be listed in the panel's hint row. Pasted text SHALL use bracketed paste
when the program in the container enables it.

#### Scenario: Interrupt

- **WHEN** a long-running command is executing in the session and the user presses Ctrl-C
- **THEN** the interrupt reaches the container and the command stops, and no application command runs

#### Scenario: Letter keys that are app shortcuts elsewhere

- **WHEN** the user types `d`, `y`, `s` or `l` in a running session
- **THEN** the characters reach the container and no Fernrohr command runs

#### Scenario: Tab completion

- **WHEN** the user types a partial path and presses Tab in a shell that supports completion
- **THEN** the shell completes it, and focus does not leave the panel

### Requirement: Terminal size follows the panel

The application SHALL size the terminal grid to the panel's area using the current font metrics.
Whenever the panel is resized, or the text size changes, it SHALL send the new size to the
container's TTY.

#### Scenario: Resize while a full-screen program runs

- **WHEN** `top` is running and the user splits the group so the panel narrows
- **THEN** `top` redraws to the new width without wrapping artifacts

### Requirement: Scrollback, selection and copy

The terminal SHALL keep a scrollback buffer, at least 10,000 lines, that can be scrolled with the
mouse wheel, the trackpad and keyboard shortcuts. It SHALL support selecting text with the mouse
and copying it with the platform copy shortcut. When the program in the container has enabled
mouse reporting, mouse input SHALL go to the container instead.

#### Scenario: Copy output

- **WHEN** the user selects lines of earlier output with the mouse and presses the platform copy shortcut
- **THEN** the selected text is on the clipboard without terminal control codes

### Requirement: Ended sessions stay readable

When the exec session ends, because the shell exited, the container stopped or the connection
dropped, the panel SHALL stop accepting input, keep the final screen and scrollback visible and
selectable, and show an ended notice that includes the exit status when it is known.

#### Scenario: Shell exits

- **WHEN** the user types `exit` in the session
- **THEN** the panel shows the session as ended with the exit status, and the earlier output is still visible and can be copied
