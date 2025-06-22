# `packages/opencode/src/app/app.ts`

## Overview

This file defines the `App` namespace, which is responsible for managing the application's context, state, and services within the OpenCode environment. It handles the creation of an application instance based on the current working directory, detects Git repository context, manages paths, and provides a way to register and use services with lifecycle management (initialization and shutdown).

## Key Components

*   **`App.Info` (Zod Schema & Type):** Defines the structure for application information. This includes:
    *   `user`: The current username.
    *   `git`: A boolean indicating if the application is running within a Git repository.
    *   `path`: An object containing various relevant paths:
        *   `config`: Global configuration path.
        *   `data`: Project-specific or global data path.
        *   `root`: The root path of the project (either Git root or current working directory).
        *   `cwd`: The current working directory from which the app was initialized.
        *   `state`: Global state path.
    *   `time`: An object for time-related information, like `initialized` (timestamp of app initialization).
*   **`create(input: { cwd: string })` (async function):**
    *   The core function for creating an application context.
    *   Determines if the `cwd` is within a Git repository using `Filesystem.findUp(".git", ...)`.
    *   Constructs the project-specific data path (or a global path if not in a Git repo).
    *   Reads/writes an `app.json` file in the data path to store/retrieve state like the initialization timestamp.
    *   Initializes an empty `services` map to hold application-specific services.
    *   Populates and returns an object containing the `services` map and the `info` (App.Info) object.
*   **`state<State>(key: any, init: (app: Info) => State, shutdown?: (state: Awaited<State>) => Promise<void>)` (function):**
    *   A factory function for creating and managing singleton services or stateful components within the app context.
    *   When the returned function is called:
        *   It retrieves the current app context using `ctx.use()`.
        *   If the service (identified by `key`) hasn't been initialized yet, it calls the `init` function (passing `app.info`) and stores the resulting state and the optional `shutdown` function.
        *   Returns the initialized service/state.
*   **`info()` (function):**
    *   Returns the `App.Info` object from the current application context.
*   **`provide<T>(input: { cwd: string }, cb: (app: Info) => Promise<T>)` (async function):**
    *   The main entry point for running code within an application context.
    *   Calls `create(input)` to set up a new application instance.
    *   Uses `ctx.provide(app, ...)` to make this app instance available via `ctx.use()` within the callback `cb`.
    *   After `cb` completes, it iterates through all registered services and calls their `shutdown` functions if provided.
    *   Returns the result of the `cb`.
*   **`initialize()` (async function):**
    *   Updates the `initialized` timestamp in the `App.Info` object and writes it to the `app.json` file in the project's data directory. This marks the app as "initialized".

## Important Variables/Constants

*   **`log` (Log instance):** A logger specific to the "app" service.
*   **`ctx` (Context instance):** A context object created using `Context.create()`, used to manage and provide access to the current application instance (`Awaited<ReturnType<typeof create>>`).
*   **`APP_JSON` (constant):** The name of the file ("app.json") used to store persistent application state (like initialization time) within the project's data directory.
*   **`Global.Path.data`, `Global.Path.config`, `Global.Path.state`:** Paths from the `Global` module used to construct various application-related paths.

## Usage Examples

**Providing an app context to run operations:**
```typescript
import { App } from "./app";
import { SomeService } from "./some-service";

async function main() {
  await App.provide({ cwd: process.cwd() }, async (appInfo) => {
    console.log("App Root:", appInfo.path.root);
    const serviceInstance = SomeService.use(); // Assuming SomeService uses App.state
    await serviceInstance.doSomething();
  });
}

main();
```

**Defining and using a stateful service:**
```typescript
// in some-service.ts
import { App } from "./app";

interface MyServiceState {
  counter: number;
  increment: () => void;
}

async function shutdownMyService(state: MyServiceState) {
  console.log("MyService shutting down, counter was:", state.counter);
}

export const useMyService = App.state<MyServiceState>(
  "myService", // Unique key
  (appInfo) => { // Init function
    console.log("Initializing MyService for user:", appInfo.user);
    return {
      counter: 0,
      increment() { this.counter++; }
    };
  },
  shutdownMyService // Optional shutdown function
);

// elsewhere
const myService = useMyService();
myService.increment();
```

## Dependencies and Interactions

*   **`../util/log` (`Log`):** Used for logging within the App namespace.
*   **`../util/context` (`Context`):** Used to create and manage the scoped application context.
*   **`../util/filesystem` (`Filesystem`):** Used to find the `.git` directory (`Filesystem.findUp`).
*   **`../global` (`Global`):** Provides global path configurations.
*   **`path` (Node.js module):** Used for path manipulations.
*   **`os` (Node.js module):** Used to get user information (`os.userInfo()`).
*   **`zod`:** Used for schema definition and validation of `App.Info`.
*   **`Bun` runtime:** Implicitly uses `Bun.file` and `Bun.write` for file operations with `app.json`.
*   **Services/Modules using `App.state`:** Any part of the application that defines services or stateful components using `App.state` will depend on this module for initialization and access to app information.
*   **`packages/opencode/src/index.ts`:** The main CLI entry point uses `App.provide` to set up the application environment before launching the TUI or executing commands.

---
*Generated by OpenCode AI Agent.*
