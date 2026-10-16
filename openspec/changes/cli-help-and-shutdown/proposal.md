# Proposal

## Why

`fernrohr` has exactly one recognized argument today: `mcp` as `argv[1]`
(`App/app/src/mcp/entry.rs`). Anything else, `--help` included, falls through to `main`'s full app
startup. It sweeps stale tunnel pidfiles, installs the app log, and opens and activates a window.
An agent probing `fernrohr --help` to learn how to wire the MCP server gets a second Fernrohr in
front of the user instead of usage text. The pidfile sweep can also reap the forwards of the app
instance that is already running.

The `fernrohr mcp` stdio adapter also outlives its client. Users see leftover `fernrohr mcp`
processes after Claude Code shuts down (App#253). The adapter builds a current-thread tokio
runtime and blocks on rmcp's stdio transport. It has no signal handling, and `tokio::io::stdin`
reads on a blocking-pool thread. Dropping the runtime waits for that thread, and it never returns
while nobody writes to stdin. So a client that goes away without closing the pipe cleanly, or a
grandchild that still holds the pipe open, leaves the adapter running forever.

## What Changes

- **Help and version.** `fernrohr --help`, `-h` and `help` print usage to stdout and exit 0.
  `fernrohr --version` and `-V` print the version and exit 0. `fernrohr mcp --help` prints the MCP
  command's usage without connecting to the app. None of them start the app, sweep pidfiles, write
  the app log or touch any window.
- **Unrecognized invocations do nothing visible.** Any argument the app doesn't recognize, an
  unknown subcommand or flag, or an extra argument after `mcp`, prints an error and the usage to
  stderr and exits 2. It doesn't start or bring forward any app instance. The app starts only with
  no arguments. On macOS it also tolerates the launcher-supplied `-psn_*` argument.
- **The MCP adapter exits with its client.** `fernrohr mcp` exits promptly, without waiting on its
  blocking stdin reader, when any of these happen:
  - its stdin reaches EOF or errors;
  - it receives SIGTERM, SIGINT or SIGHUP;
  - its parent process goes away (it is reparented).

  In-flight requests to the app are abandoned. The app side already treats each request as its
  own socket connection, so it needs no change.

## Capabilities

### New Capabilities
- `command-line`: what the `fernrohr` executable does with its arguments: help, version,
  rejecting unrecognized invocations, and starting the app only when it has no arguments.

### Modified Capabilities
- `agent-mcp`: adds a requirement that the stdio MCP command exits when its client goes away.

## Impact

- `App/app/src/mcp/entry.rs` grows into a small hand-rolled argument dispatcher, or a new
  `cli.rs` takes it over. No clap: the surface is a handful of words.
- `App/app/src/main.rs`: the app path runs only when the dispatcher says "start the app".
- `App/app/src/mcp/adapter.rs`: shutdown select over stdin EOF, signals and parent watch, then
  `Runtime::shutdown_background` (or `shutdown_timeout`) before exiting.
- `App/app/Cargo.toml`: tokio `signal` feature.
- `App/docs/site/latest/agents.md`: mention `fernrohr --help` / `fernrohr mcp --help`.
- No change to the packaged launchers. The `.desktop` entry is `Exec=fernrohr` and the macOS
  bundle passes no arguments.
