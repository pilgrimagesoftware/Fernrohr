## ADDED Requirements

### Requirement: Help and version output
The `fernrohr` executable SHALL print its usage to standard output and exit with status 0 when invoked with `--help`, `-h` or `help`, and SHALL print its version and exit with status 0 when invoked with `--version` or `-V`. `fernrohr mcp --help` SHALL print the MCP command's usage without connecting to the running app. None of these invocations SHALL start the app, write the app log, alter tunnel state, or open, activate or focus any window.

#### Scenario: Agent asks for help
- **WHEN** an agent runs `fernrohr --help` while Fernrohr is running in the background
- **THEN** the usage is printed, the process exits with status 0, and the running Fernrohr stays in the background with its tunnels untouched

#### Scenario: Help for the MCP command
- **WHEN** `fernrohr mcp --help` is run while no Fernrohr is running
- **THEN** the MCP command's usage is printed and the process exits with status 0 without reporting the app as unavailable

#### Scenario: Version
- **WHEN** `fernrohr --version` is run
- **THEN** the version is printed and the process exits with status 0

### Requirement: Unrecognized invocations start nothing
The `fernrohr` executable SHALL start the app only when invoked with no arguments, apart from arguments the platform's own launcher supplies. Any other unrecognized argument, unknown subcommand, or extra argument after a subcommand SHALL print an error and the usage to standard error and exit with status 2, without starting the app, writing the app log, altering tunnel state, or activating any window.

#### Scenario: Unknown flag
- **WHEN** `fernrohr --verbose` is run while Fernrohr is running
- **THEN** an error naming `--verbose` and the usage are printed to standard error, the process exits with status 2, and the running Fernrohr does not come to the front

#### Scenario: Extra argument after mcp
- **WHEN** `fernrohr mcp serve` is run
- **THEN** the process exits with status 2 after printing an error, and starts neither the MCP adapter nor the app

#### Scenario: Normal launch
- **WHEN** Fernrohr is started from the desktop launcher or as `fernrohr` with no arguments
- **THEN** the app starts as before
