# `packages/opencode/src/index.ts`

## Overview

This file serves as the main entry point for the OpenCode CLI application. It initializes the application, sets up command-line argument parsing using `yargs`, defines available commands, and handles the primary application flow, including launching the Textual User Interface (TUI). It also manages logging, error handling, and an auto-update mechanism.

## Key Components

*   **`cli` (yargs instance):** The core `yargs` instance configured for the `opencode` command.
    *   Sets the script name to `opencode`.
    *   Configures version information using `Installation.VERSION`.
    *   Adds a global `--print-logs` option.
    *   Includes middleware for initializing `Log` and logging basic invocation details.
    *   Displays a logo using `UI.logo()` in usage instructions.
*   **Default Command (`$0 [project]`):**
    *   Describes the main action: starting the OpenCode TUI.
    *   Takes an optional `project` positional argument for the directory.
    *   **Handler Logic:**
        1.  Continuously attempts to run the app until successful or explicitly broken.
        2.  Resolves the current working directory (`cwd`).
        3.  Uses `App.provide` to set up the application context.
        4.  Checks for available providers (`Provider.list()`). If none, it triggers the `AuthLoginCommand` flow.
        5.  Initializes `Share` functionality.
        6.  Starts the local `Server` (`Server.listen()`) for the TUI to connect to.
        7.  Determines the TUI binary path (either from embedded files if available, or by running `go run` for local development).
        8.  Spawns the TUI process (`Bun.spawn`) with necessary environment variables (`OPENCODE_SERVER`, `OPENCODE_APP_INFO`).
        9.  Initiates an asynchronous auto-update check (`Installation.latest()`, `Installation.upgrade()`) unless in dev mode, snapshot, or auto-update is disabled.
        10. Waits for the TUI process to exit and then stops the server.
        11. If providers were missing, it prompts the user to log in via `AuthLoginCommand`.
*   **Other Commands:**
    *   `RunCommand`: Likely for running specific tasks or scripts.
    *   `GenerateCommand`: Potentially for code generation.
    *   `ScrapCommand`: Could be for web scraping or data extraction.
    *   `AuthCommand`: Main command for authentication-related actions.
    *   `UpgradeCommand`: For manually triggering application updates.
*   **Error Handling:**
    *   `.fail()`: Custom yargs failure handler to show help for specific argument errors.
    *   `try...catch` block: Catches errors during CLI parsing and execution, logs them (including `NamedError` details), and displays a formatted error message using `UI.error()` or a generic one with the log file path.

## Important Variables/Constants

*   **`process.argv`:** Used to get command-line arguments for `yargs` and for the `--print-logs` check.
*   **`Installation.VERSION`:** Provides the current version of the application.
*   **`Global.Path.cache`:** Path used for storing the TUI binary if extracted from embedded files.
*   **`OPENCODE_SERVER` (env var for TUI):** The URL of the local server that the TUI connects to.
*   **`OPENCODE_APP_INFO` (env var for TUI):** JSON stringified application information passed to the TUI.

## Usage Examples

The script is executed via the `opencode` command:

```bash
opencode
```
This starts the TUI in the current directory.

```bash
opencode /path/to/project
```
This starts the TUI in the specified project directory.

```bash
opencode --print-logs
```
Starts the TUI and prints detailed logs to stderr.

```bash
opencode run <args...>
```
Executes the `run` command.

```bash
opencode auth login
```
Initiates the authentication login flow.

## Dependencies and Interactions

*   **`yargs`:** Core library for command-line argument parsing and handling.
*   **`./app/app` (`App`):** Used for application context management (`App.provide`).
*   **`./server/server` (`Server`):** Used to start and stop the local HTTP server for TUI communication.
*   **`./share/share` (`Share`):** Initialized for sharing functionalities.
*   **`./global` (`Global`):** Provides global path configurations.
*   **`./cli/cmd/*`:** Imports various command modules (e.g., `RunCommand`, `AuthCommand`).
*   **`./util/log` (`Log`):** Used for application-wide logging.
*   **`./cli/ui` (`UI`):** Provides UI elements like the logo and error formatting.
*   **`./installation` (`Installation`):** Handles version information, update checks, and upgrade processes.
*   **`./bus` (`Bus`):** Used for publishing events, like `Installation.Event.Updated`.
*   **`./config/config` (`Config`):** Used to check global configuration like `autoupdate` settings.
*   **`./util/error` (`NamedError`, `FormatError`):** For custom error handling and formatting.
*   **`Bun` runtime:** Uses `Bun.spawn` to run the TUI process and `Bun.file`, `Bun.write` for managing the embedded TUI binary.
*   **Go TUI:** The script is responsible for launching the Go-based TUI application (either from source or a pre-compiled binary).

---
*Generated by OpenCode AI Agent.*
