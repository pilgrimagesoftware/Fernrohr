# Design

## Context

Today the shell panel (`k8s/resource/exec/`) opens a kube-rs exec session, renders output as a log
and sends whole lines from an input field. Knot's terminal, `crates/knot-terminal` plus
`crates/knot/src/terminal_view.rs`, wraps `alacritty_terminal` 0.26 behind a `Grid`, encodes keys,
mouse input and pastes into terminal bytes, and renders the grid in GPUI. The Knot agents are
extracting it into an MIT-licensed `gpui-terminal` crate with a transport trait (write bytes,
receive output, resize, terminate, exit status) and a ready-made view.

## Goals / Non-Goals

**Goals:**

- Reuse the shared crate for emulation, rendering and input encoding. Fernrohr supplies only a
  kube exec transport and the panel around it.

**Non-Goals:**

- Local shells, or `kubectl debug` ephemeral containers. The latter would be a follow-up command
  that reuses this terminal.
- Recording or replaying sessions.
- Multiplexing several sessions in one panel. Each shell stays one panel.

## Decisions

### 1. Transport: kube-rs `AttachedProcess` with a TTY

Open the exec session with `AttachParams { stdin: true, stdout: true, stderr: false, tty: true }`,
since with a TTY stderr is merged into stdout, and run `sh -c` over a small probe list
(`bash || ash || sh`) so the best available shell starts, as the current bridge does. Then:

- **Output:** a tokio task reads the `stdout()` stream and forwards the bytes into the crate's
  output sink. The GPUI side receives them through the existing tokio↔GPUI bridge channel, never
  by blocking the UI thread.
- **Input:** bytes from `key_to_bytes`, `mouse_to_bytes` and `paste_payload` go to `stdin()`
  through a bounded channel.
- **Resize:** `terminal_size()` returns a sender that carries `TerminalSize { width, height }`.
  The crate's view calls `resize(rows, cols)` after layout, and the transport forwards it,
  debounced so a drag-resize sends only the final size.
- **Exit:** the `take_status()` future resolves to the exit status, which is reported to the view
  so it can show the ended notice.

### 2. Key ownership in the panel

The terminal element owns key input while the panel has focus. The panel's key context becomes
`ExecTerminal`, and nothing in the app keymap binds plain or `ctrl-` keys in that context. The
reserved shortcuts are:

- `cmd-c` and `cmd-v` on macOS, `ctrl-shift-c` and `ctrl-shift-v` elsewhere
- the command palette, `cmd-shift-p`
- panel and tab navigation, which use `cmd-` keys

`cmd-c` copies the selection when there is one, and otherwise is sent to the terminal. The panel's
existing hint row shows the reserved keys.

### 3. Theme and font

The crate's view takes a palette and a font. Fernrohr derives the palette's 16 ANSI colors, plus
foreground, background, cursor and selection, from `cx.theme()`, with dark and light variants.
The font is the app's monospace font at the current text-size scale. A text-size change triggers
a re-layout and, through decision 1, a resize.

### 4. Lifecycle

The panel keeps its current responsibilities: the container picker, offering the shell only when
a container is running, its close warning (the `close_warning()` used by Close Group), and its
key context name for tests. Releasing the panel drops the transport, which closes stdin and aborts
the reader task, as the current bridge already does.

## Risks / Trade-offs

- [The crate isn't published yet, or its GPUI or gpui-kit version differs from Fernrohr's] → this
  change depends on the extraction, and the Knot side has been asked to match gpui-kit 0.7 and
  `gpui-pre` 0.3.7. If they differ, bump them in a separate PR first.
- [Exec sessions without a TTY on old clusters] → `tty: true` is supported on every Kubernetes
  version kube-rs supports, so no fallback to the line view.
- [Taking all keys means panel shortcuts disappear while the terminal has focus] → the reserved set
  plus `cmd-` navigation keeps the panel reachable, and the hint row lists them.
