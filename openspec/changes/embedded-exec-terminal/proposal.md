# Proposal

## Why

The pod shell from `k9s-remaining-keybindings` is line-based: a command prompt with an output log.
It can run single commands, but nothing that expects a terminal works:

- full-screen programs (`vim`, `less`, `top`)
- tab completion and line editing
- Ctrl-C and other control keys
- colors and cursor movement

People who shell into a container expect a real terminal. Knot, a sibling Pilgrimage Software app,
already has a working GPUI terminal built on `alacritty_terminal`, and it is being moved into a
shared MIT-licensed crate (working name `gpui-terminal`) so Fernrohr can use it.

## What Changes

- Replace the shell panel's prompt-and-log view with an embedded terminal emulator from the shared
  `gpui-terminal` crate, connected to a Kubernetes exec session opened with a TTY.
- Every key, control sequence, mouse event and paste goes straight to the container's terminal.
  The panel's own shortcuts are limited to a small set that never collides with terminal input.
- When the panel is resized, the container's terminal size follows, so full-screen programs lay
  out correctly.
- When the session ends, the terminal's final screen and scrollback stay visible, with an "ended"
  notice. Choosing a container and offering the shell only when a container is running stay as
  they are.
- Copying the selection, scrollback, and the terminal's font and colors follow Fernrohr's theme
  and text size.

## Capabilities

### New Capabilities

- `exec-terminal`: the embedded terminal that hosts a container exec session, covering input
  pass-through, resizing, scrollback and copy, theming, and how it behaves when the session ends.

### Modified Capabilities

(none: the shell requirement in `k9s-remaining-keybindings` already says "an interactive exec
session in a panel", and this change only defines what that panel is)

## Impact

- `App/app/Cargo.toml`: a new git dependency on the shared `gpui-terminal` crate, pinned by tag
  or rev. It brings in `alacritty_terminal`.
- `App/app/src/k8s/resource/exec/`: `render.rs` is replaced by the terminal view. `bridge.rs`
  becomes the crate's transport, carrying stdin and stdout bytes and terminal resizes over a kube
  exec session with `tty: true`. `panel.rs` keeps the panel's lifecycle, its close warning and its
  key context.
- Depends on the crate extraction in the Knot repository, which the Knot agents are doing. This
  change can't be implemented until the crate is published.
