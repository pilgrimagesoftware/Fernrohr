# Tasks

## 1. Dependency

- [ ] 1.1 Add the shared `gpui-terminal` crate as a git dependency pinned to a tag or rev, and
      align the GPUI and gpui-kit versions. Verify that `cargo build` and
      `cargo tree -d -e normal` show a single gpui-kit and a single `gpui-pre`.

## 2. Transport

- [ ] 2.1 Implement the crate's transport trait over a kube-rs exec session with `tty: true`:
      - output bytes are forwarded to the view through the tokio↔GPUI bridge
      - stdin goes through a bounded channel
      - resizes are debounced and sent on the `terminal_size()` channel
      - the exit status comes from `take_status()`

      Verify with unit tests against a fake attached process: bytes round-trip, the last resize
      wins, exit is reported, and dropping the transport closes stdin and stops the reader.

## 3. Panel

- [ ] 3.1 Replace the shell panel's log view with the crate's terminal view, using the
      `ExecTerminal` key context and the reserved shortcut set. Verify with keystroke tests that
      Ctrl-C, Tab, `d`, `y`, `s` and `l` reach the transport as bytes without running app
      commands, and that `cmd-shift-p` still opens the palette.
- [ ] 3.2 Map the theme to the terminal palette, apply the font and text size, and re-layout and
      resize when text size changes. Verify with a test that a text-size change sends a new size.
- [ ] 3.3 On session end: stop accepting input, keep the screen and scrollback, and show an ended
      notice with the exit status. Keep the container picker, running-container gating and
      `close_warning()`. Verify with tests for an exit status, a dropped connection, and Close
      Group's warning while the session is running.

## 4. Gates

- [ ] 4.1 `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` all
      pass, and touched files stay under 500 lines. Update the README's shell section.

## 5. Verification

- [ ] 5.1 Manual check on a real cluster, needing user confirmation:
      - Shell into a running pod.
      - Run `top`, then narrow the panel with a split and confirm `top` redraws. Quit with `q`.
      - Open `vi`, edit, and save.
      - Use Tab completion.
      - Interrupt `sleep 100` with Ctrl-C.
      - Select and copy earlier output.
      - Run `exit`, and confirm the ended notice and exit status, with the scrollback still
        readable.
