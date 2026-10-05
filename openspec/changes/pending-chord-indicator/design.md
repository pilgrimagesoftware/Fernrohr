# Design

## Context

GPUI (`gpui-pre` 0.3.6) keeps a window's pending multi-stroke input itself:

- `Window::pending_input()` returns the keystrokes typed so far and an optional timeout status.
  Input left over from an earlier focus is filtered out.
- `cx.observe_pending_input(window, …)` fires whenever that input starts, extends, completes or
  is dropped.
- A timeout exists only when the pending keys are themselves a complete binding, or when the key
  produced a character while a text input accepts text. Its length is the crate-private
  `PENDING_INPUT_TIMEOUT` (1 s), which can't be configured.
- `Window::set_pending_input_timeout_paused(owner, paused, cx)` lets an entity pause and resume
  that timeout. A pause from an owner that is released resumes automatically.

The status bar (`ui/status_bar.rs`) already shows per-window state, such as context health.

## Goals / Non-Goals

**Goals:**

- Read and render GPUI's pending state rather than tracking keystrokes separately, so the
  indicator can never disagree with dispatch.
- Lengthen the ambiguous-case timeout without forking GPUI.

**Non-Goals:**

- Changing how GPUI matches or replays keys.
- A timeout for unambiguous prefixes. GPUI never times them out, and adding one would only make
  shortcuts fail more often.
- An Escape key that cancels a pending shortcut explicitly. Pressing any non-completing key
  already abandons it.

## Decisions

### 1. Indicator is driven by `observe_pending_input`

`StatusBar` subscribes with `cx.observe_pending_input(window, …)`. On each callback it reads
`window.pending_input()`, stores the keystrokes, or clears them when the result is `None`, and
calls `cx.notify()`. Because `pending_input()` filters out input from a previous focus, a focus
change clears the indicator without extra code.

- **Alternative: a global keystroke observer that reimplements prefix matching.** Rejected, because
  it would duplicate GPUI's matcher, including context predicates and replay, and could drift from
  it.

### 2. Completion list comes from the effective keymap

Add a keymap query: given the pending keystrokes and the window's current context stack
(`window.context_stack()`), return each binding whose keystrokes start with the pending ones and
whose context predicate matches. Each result gives the remaining keys and the command id, which
the command registry resolves to a title. Use the effective keymap, defaults plus `keymap.toml`,
so a rebound chord shows its new keys. Sort with the palette's own ordering. Render keys with the
palette's existing key-notation helper.

The list renders as a compact popover anchored to the status bar indicator, above the bar, and
never focusable. If more completions exist than fit (8 rows), it ends with "… and N more". Arrange
commands produce 13 entries, so in practice the list holds the five `cmd-k` completions, since the
rest are different first keys.

### 3. Timeout extension pauses GPUI's timer and runs our own

When a callback shows a pending input that has a timeout, `StatusBar` pauses GPUI's timeout with
`set_pending_input_timeout_paused(&status_bar_entity, true, cx)` and spawns a timer for
`shortcut_timeout - PENDING_INPUT_TIMEOUT`, at least zero. When that timer fires, it resumes GPUI's
timeout, which then runs out its remaining time and flushes the shorter binding through GPUI's own
path. The total wait is about the configured value. If the pending input changes or clears first,
the timer task is dropped and the pause is released.

The pause is tied to the status bar entity, so a window closing mid-chord resumes the timeout
automatically. The 1 s constant is mirrored as `GPUI_PENDING_INPUT_TIMEOUT` with a comment and a
test that pins total duration, so a GPUI bump that changes it gets noticed.

- **Alternative: patch `gpui-pre` to make the constant configurable.** Rejected for now, because
  the public pause API is meant for exactly this. Note it as a possible upstream request.

### 4. Preference

Add `UiConfig.shortcut_timeout_secs: ShortcutTimeout(u8)`. It uses a clamping `From<u8>` in the
`TextSize` style: default 3, range 1–10, serialized as a bare integer. The Settings panel gets a
stepper row next to text size. `StatusBar` reads it from the live `UiConfig` global when it starts
each timer, so changes apply immediately.

## Risks / Trade-offs

- [GPUI's pause semantics change in a future release] → timing tests on the extension, plus the
  mirrored constant noted above.
- [The popover covers content at the bottom of a panel for a few seconds] → it shows only while a
  chord is pending, sits above the status bar and is small.
- [Ambiguous bindings are rare today: none of the defaults are] → the timeout preference mostly
  protects user-made keymaps. That is still worth it, because the keybindings editor makes such
  keymaps easy to create.
