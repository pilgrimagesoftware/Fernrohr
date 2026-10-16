# Design

## Context

`main` asks `mcp::run_subcommand(args_os)` first. `Some(code)` exits, and `None` starts the full
app. The adapter (`mcp/adapter.rs`) runs `runtime.block_on(serve(client, rmcp::transport::stdio()))`
on a `new_current_thread` runtime. `serve` awaits `running.waiting()`, which returns once rmcp's
transport closes.

## Decisions

### D1: One dispatcher, three outcomes

A pure function `parse(args) -> Invocation` with `StartApp`, `Mcp`, `Print { text, to_stderr,
code }` keeps every case unit-testable without spawning a process. `main` matches on it.
`Invocation::StartApp` is returned only for no arguments (macOS: ignoring `-psn_*`). Recognized
words are `help`, `-h`, `--help`, `-V`, `--version` and `mcp`. Under `mcp`, only `-h` and
`--help` are allowed. Everything else is `Print { to_stderr: true, code: 2 }` with a one-line
error followed by the usage.

Rejected: treating unknown arguments as "start the app anyway" (the issue is exactly that), and
adding clap (a large dependency for six words; the help text stays hand-written and short).

### D2: Print paths run before anything else in `main`

Help, version and errors return before `util::pidfile::sweep_stale()`, `app_log::install()` and
GPUI startup. The existing `mcp` early return already relies on this ordering. The dispatcher
keeps it, so only `StartApp` can reach the sweep.

### D3: Adapter shutdown is a select, then a background shutdown

`serve` races `running.waiting()` against a shutdown future:
- `tokio::signal::unix` streams for SIGTERM, SIGINT and SIGHUP;
- a parent watch: record `getppid()` at start, poll it every `consts::MCP_PARENT_POLL`
  (1 s), and fire when it changes.

On a signal or parent loss it cancels the rmcp service. `run()` then calls
`runtime.shutdown_background()`, rather than letting the runtime drop, so the blocking stdin reader
can't hold the process. `main` then calls `std::process::exit` as before. Exit code 0 for EOF and
parent loss, 128+signo for signals.

Stdin EOF needs no new code beyond the background shutdown. rmcp already ends `waiting()`, and
what hung was the runtime drop.

### D4: Help text content

Usage lists `fernrohr` (start the app), `fernrohr mcp` (the stdio MCP server an agent's client
launches; needs a running Fernrohr), `--help` and `--version`. The help text has no hostnames or
paths. `fernrohr mcp --help` adds that it talks to the running app over a local socket and never
starts a window.

## Risks

- A future launcher that passes arguments (file associations, URL schemes) would now be rejected.
  That is intended: add it to the dispatcher when it exists.
- Parent polling costs one syscall a second while an agent session is open, which is negligible.
