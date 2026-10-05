# Tasks

## 1. Keymap completion query

- [x] 1.1 Add a keymap function that, given pending keystrokes and a context stack, returns each
      effective binding whose keystrokes extend them and whose context matches, as remaining
      keys, command id and title, in palette order. Verify with unit tests for:
      - the `cmd-k` arrange chords
      - a rebound chord (`cmd-k h`)
      - a context that excludes the bindings (`!Input`)
      - an empty result for a non-prefix

## 2. Status bar indicator

- [x] 2.1 Subscribe `StatusBar` to `observe_pending_input`. Render the pending keys with an
      ellipsis, and a non-focusable completion popover capped at 8 rows with "… and N more".
      Clear on `None`. Verify with GPUI keystroke tests:
      - `cmd-k` shows the indicator and five completions
      - `cmd-k left` clears it and splits
      - `cmd-k x` clears it and runs nothing
      - a focus change clears it
      - a single-step shortcut never shows it
- [x] 2.2 Confirm the indicator never takes focus and doesn't change dispatch. Verify with a test
      that the completing key reaches the same action with the status bar present and absent.

## 3. Shortcut timeout

- [x] 3.1 Add `ShortcutTimeout` (default 3, clamped 1–10, bare integer) to `UiConfig`. Verify
      with round-trip and clamping tests, including that an out-of-range value keeps the other
      fields.
- [x] 3.2 Add a Settings row with a keyboard-operable stepper. Verify with a settings view test
      that changing it persists to `ui.toml`.
- [x] 3.3 Implement the pause-and-extend timer in `StatusBar`, as in design decision 3. Verify
      with time-controlled tests for an ambiguous binding: the shorter binding fires at about the
      configured duration (3 s default, 6 s after changing it) and not before. A pending input
      that changes mid-wait cancels the timer, and a closed window releases the pause.
- [x] 3.4 Confirm unambiguous prefixes never time out. Verify with a test that `cmd-k`, then a
      wait longer than the timeout, then `w` still runs Close Group.

## 4. Gates and documentation

- [x] 4.1 Document the indicator and the timeout preference in the user docs' keyboard section.
      `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` all
      pass, and touched files stay under 500 lines.

## 5. Verification

- [ ] 5.1 Manual check on a running build. **Needs user confirmation.**
      1. Focus a panel and press `cmd-k`. The status bar shows `⌘K …`, and above it five rows:
         the four arrows (Split Group Left/Right/Up/Down) and `w` (Close Group).
      2. Press `←`. The group splits, and the indicator and list disappear.
      3. Press `cmd-k`, then `x`. The indicator disappears, and nothing splits or closes.
      4. Press `cmd-k`, wait ten seconds, then press `w`. The indicator stays the whole time, and
         the group closes.
      5. In Settings → Keyboard Shortcuts, set Focus Resources to `cmd-k`, and accept the
         warning that it starts other commands' chords (they keep their keys). Focus a panel and
         press `cmd-k` alone. The indicator shows,
         and focus moves to the Resource panel about 3 seconds later, not after 1.
      6. In Settings, set Shortcut Timeout to 6 s, then repeat step 5. Focus moves after about
         6 seconds. Reset Focus Resources to its default afterwards.
