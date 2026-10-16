# Tasks

## 1. Argument dispatcher

- [x] 1.1 Replace `mcp::run_subcommand` with a pure `cli::parse(args) -> Invocation`
  (`StartApp`, `Mcp`, `Print { text, to_stderr, code }`) per design D1, with the usage and MCP
  usage text from D4. Tolerate `-psn_*` on macOS only. Verify with unit tests covering every
  scenario in `specs/command-line`: `--help`, `-h`, `help`, `--version`, `-V`, `mcp --help`, an
  unknown flag, an unknown subcommand, `mcp serve`, no arguments and `-psn_0_123` on macOS.
  Assert the exit code, the stream and that the error names the offending argument. Replace
  `only_the_mcp_subcommand_skips_the_app`.
- [x] 1.2 Rewire `main` so `Print` writes and exits and `Mcp` runs the adapter, both before
  `sweep_stale`, `app_log::install` and GPUI startup (D2). Verify with an integration test that
  runs the built binary (`env!("CARGO_BIN_EXE_fernrohr")`) with `--help` and `--bogus`, with a
  fresh `HOME`/XDG dirs. Assert the exit codes and output, and that no app log file and no data
  or preference dir was created.

## 2. MCP adapter shutdown

- [x] 2.1 Enable tokio's `signal` feature. In `adapter::serve`, race `running.waiting()` against
  SIGTERM/SIGINT/SIGHUP and a parent watch (`getppid` change, polled every
  `consts::MCP_PARENT_POLL`), and cancel the service on either (D3). Verify with an in-memory
  transport test: firing an injected shutdown future ends `serve`.
- [x] 2.2 End `adapter::run` with `runtime.shutdown_background()` and the exit codes from D3.
  Verify with integration tests on the built binary that cover the three `agent-mcp`
  "MCP command exits with its client" scenarios:
  - close stdin and assert exit within 1 s;
  - send SIGTERM with stdin held open and assert exit within 1 s;
  - spawn it through an intermediate process that exits and leaves stdin open, and assert exit
    within 2 s.

  None needs a running app.

## 3. Documentation

- [x] 3.1 In `docs/site/latest/agents.md`, note that `fernrohr --help` and `fernrohr mcp --help`
  print usage without opening the app, and that the MCP command ends when the agent's client
  closes it. Verify with the docs build and link check.
