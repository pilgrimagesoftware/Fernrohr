# Spec Delta

## Purpose

Defines where Fernrohr writes its application log, how long a run's log is kept, and which records
it contains, so a user can attach it to a bug report.

## ADDED Requirements

### Requirement: Fernrohr writes an application log file

At startup the application SHALL write its log records to `fernrohr.log` in a per-user log
directory: `~/Library/Logs/com.pilgrimagesoftware.fernrohr/` on macOS, and
`$XDG_STATE_HOME/fernrohr/` on Linux (the cache directory when no state directory exists). Records
SHALL carry a timestamp, a level and the module that wrote them.

#### Scenario: The app starts

- **WHEN** Fernrohr starts
- **THEN** it creates the log directory if needed and writes its records to `fernrohr.log` there

#### Scenario: The log can't be opened

- **WHEN** the log file can't be created or opened
- **THEN** Fernrohr starts normally without a log file

### Requirement: The previous run's log is kept

At startup the application SHALL rename an existing `fernrohr.log` to `fernrohr.log.1`, replacing
any older `fernrohr.log.1`, before it writes the new run's log.

#### Scenario: A second run

- **WHEN** Fernrohr starts and a `fernrohr.log` exists from the last run
- **THEN** that file becomes `fernrohr.log.1` and the new run starts an empty `fernrohr.log`

### Requirement: The log level defaults to info

The application SHALL record `info` and more severe records, unless `RUST_LOG` names one of
`error`, `warn`, `info`, `debug` or `trace`, in which case that level SHALL apply. When stderr is a
terminal, the records SHALL also be written to stderr.

#### Scenario: Default level

- **WHEN** Fernrohr starts without `RUST_LOG`
- **THEN** `info`, `warn` and `error` records are written and `debug` and `trace` records are not

#### Scenario: Running from a terminal

- **WHEN** Fernrohr runs with stderr attached to a terminal, as under `cargo run`
- **THEN** each record is written to stderr as well as to the log file
