## ADDED Requirements

### Requirement: MCP command exits with its client
The stdio MCP command SHALL exit promptly, abandoning any in-flight request, when its standard input reaches end of file or fails, when it receives SIGTERM, SIGINT or SIGHUP, or when its parent process exits. Exiting SHALL NOT wait for a pending read of standard input.

#### Scenario: Client closes the pipe
- **WHEN** the MCP client closes the command's standard input
- **THEN** the command exits within one second

#### Scenario: Client terminates the command
- **WHEN** the MCP client sends SIGTERM to the command while its standard input is still open
- **THEN** the command exits within one second

#### Scenario: Client dies without cleanup
- **WHEN** the process that launched the MCP command exits without closing the command's standard input or signaling it
- **THEN** the command exits within two seconds
