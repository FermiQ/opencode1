# `packages/tui/cmd/opencode/main.go`

## Overview

This Go program is the main entry point for the OpenCode Textual User Interface (TUI). It initializes the application environment, sets up logging, establishes communication with the OpenCode backend server (typically running locally and started by the `packages/opencode/src/index.ts` script), and launches the Bubble Tea TUI application.

## Key Components

*   **`Version` (variable):** A global variable that holds the version string of the TUI. This is typically set during the build process.
*   **`main()` (function):**
    1.  **Version Handling:** Normalizes the `Version` string, prefixing with "v" if it's not "dev" and doesn't already have it.
    2.  **Environment Variables:**
        *   Reads `OPENCODE_SERVER`: The URL of the backend server.
        *   Reads `OPENCODE_APP_INFO`: A JSON string containing application information (paths, user, etc.) provided by the Node.js part of the CLI. This is unmarshalled into `client.AppInfo`.
    3.  **Logging Setup:**
        *   Constructs a log file path: `appInfo.Path.Data + "/log/tui.log"`.
        *   Ensures the log directory exists.
        *   Creates/opens the log file.
        *   Sets up `slog` (structured logger) to write debug-level logs to this file.
    4.  **HTTP Client Creation:**
        *   Creates an HTTP client (`client.NewClientWithResponses(url)`) to interact with the `OPENCODE_SERVER`. This client is generated from an OpenAPI specification and includes methods for all defined API endpoints.
    5.  **Application Context:** Creates a Go `context.Context` with cancellation (`context.WithCancel`).
    6.  **App Initialization:**
        *   Creates a new instance of the TUI application logic (`app.New(ctx, version, appInfo, httpClient)`). This `app` instance likely holds state and methods for interacting with the backend.
    7.  **Bubble Tea Program:**
        *   Initializes a new Bubble Tea program (`tea.NewProgram`) with:
            *   The main TUI model: `tui.NewModel(app_)`.
            *   Standard Bubble Tea options: `tea.WithAltScreen()`, `tea.WithKeyboardEnhancements()`, `tea.WithMouseCellMotion()`.
    8.  **Event Subscription:**
        *   Creates another client (`client.NewClient(url)`) specifically for subscribing to Server-Sent Events (SSE) from the backend's `/event` endpoint.
        *   Subscribes to the event stream (`eventClient.Event(ctx)`).
        *   Launches a goroutine that listens for incoming events on the `evts` channel and sends them to the Bubble Tea program (`program.Send(item)`), allowing the TUI to react to real-time backend events.
    9.  **Run TUI:** Runs the Bubble Tea program (`program.Run()`). This blocks until the TUI exits.
    10. **Log Exit:** Logs any error from `program.Run()` and an informational message that the TUI has exited.

## Important Variables/Constants

*   **`Version` (string):** Stores the build version of the TUI.
*   **`OPENCODE_SERVER` (environment variable):** Crucial for the TUI to know where to send API requests. Expected to be set by the process that launches this Go program (i.e., `packages/opencode/src/index.ts`).
*   **`OPENCODE_APP_INFO` (environment variable):** Provides essential application metadata (like data paths for logging) from the launcher.
*   **`httpClient` (`*client.ClientWithResponses`):** The primary client for making API calls to the backend.
*   **`eventClient` (`*client.Client`):** A client used for establishing the SSE connection for real-time events.
*   **`program` (`*tea.Program`):** The instance of the Bubble Tea TUI application.
*   **`app_` (`*app.App`):** The core application logic handler for the TUI.

## Usage Examples

This program is not typically run directly by an end-user. It is compiled into an executable and launched by the main `opencode` CLI script (`packages/opencode/src/index.ts`).

The `opencode` script sets the required environment variables (`OPENCODE_SERVER`, `OPENCODE_APP_INFO`) before executing this TUI program.

## Dependencies and Interactions

*   **`os`:** For accessing environment variables and file system operations.
*   **`log/slog`:** For structured logging.
*   **`encoding/json`:** For unmarshalling `OPENCODE_APP_INFO`.
*   **`path/filepath`:** For constructing file paths.
*   **`context`:** For managing request lifecycles and cancellation.
*   **`github.com/charmbracelet/bubbletea/v2`:** The TUI framework used to build the interactive interface.
*   **`github.com/sst/opencode/internal/app`:** Contains the core application logic for the TUI, including state management and interaction with the backend client.
*   **`github.com/sst/opencode/internal/tui`:** Contains the Bubble Tea model definitions (views, updates, commands) for the TUI.
*   **`github.com/sst/opencode/pkg/client`:** Contains the generated Go client library for interacting with the OpenCode backend server's API. This client is generated based on an OpenAPI specification.
*   **OpenCode Backend Server (Node.js):** This TUI application is a client to the server defined in `packages/opencode/src/server/server.ts`. It relies on this server for all its data and operations.
*   **`packages/opencode/src/index.ts` (Launcher):** This Node.js script is responsible for starting the backend server and then launching this Go TUI application, providing it with the necessary environment variables.

---
*Generated by OpenCode AI Agent.*
