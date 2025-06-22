# `packages/function/src/api.ts`

## Overview

This file defines a Cloudflare Worker that includes a Durable Object `SyncServer` and various HTTP endpoints for managing and synchronizing shared session data. It handles WebSocket connections for real-time updates, creation and deletion of shares, and data retrieval for shared sessions.

## Key Components

*   **`Env` (type):** Defines the expected environment bindings for the worker, including `SYNC_SERVER` (Durable Object namespace) and `Bucket` (R2 bucket).
*   **`SyncServer` (class - Durable Object):**
    *   Manages the state and communication for a specific shared session.
    *   `constructor(ctx: DurableObjectState, env: Env)`: Initializes the Durable Object with its state and environment.
    *   `async fetch()`: Handles incoming WebSocket upgrade requests. It accepts the WebSocket, sends existing session data to the new client, and establishes the connection.
    *   `async webSocketMessage(ws, message)`: Placeholder for handling incoming WebSocket messages (currently does nothing).
    *   `async webSocketClose(ws, code, reason, wasClean)`: Handles WebSocket closures.
    *   `async publish(secret: string, key: string, content: any)`: Publishes new data to all connected WebSocket clients for the session and stores it in R2 and Durable Object storage. Requires a valid secret.
    *   `async share(sessionID: string)`: Initializes a share for a given `sessionID`, generating a unique secret for it. Stores the secret and `sessionID` in Durable Object storage.
    *   `async getData()`: Retrieves all session-related data stored within the Durable Object.
    *   `async clear(secret: string)`: Deletes all data for the session from Durable Object storage. Requires a valid secret.
    *   `static shortName(id: string)`: Utility function to generate a short (last 8 characters) identifier from a given ID, used for Durable Object naming.
*   **Default Export (Worker Fetch Handler):**
    *   `async fetch(request: Request, env: Env, ctx: ExecutionContext)`: The main entry point for HTTP requests to the worker.
    *   Routes requests based on HTTP method and path:
        *   `GET /`: Returns a "Hello, world!" message.
        *   `POST /share_create`: Creates a new share for a session, returning a secret and a share URL. It gets or creates a `SyncServer` Durable Object instance based on a short name derived from the `sessionID`.
        *   `POST /share_delete`: Deletes a shared session's data. Requires `sessionID` and `secret`.
        *   `POST /share_sync`: Pushes data to a shared session. Requires `sessionID`, `secret`, `key`, and `content`. The `SyncServer` instance then publishes this data.
        *   `GET /share_poll?id=<share_id>`: Handles WebSocket upgrade requests for a specific share ID. It forwards the request to the appropriate `SyncServer` instance's `fetch()` method.
        *   `GET /share_data?id=<share_id>`: Retrieves the data (info and messages) for a specific share ID from the corresponding `SyncServer` instance.

## Important Variables/Constants

*   **`env.SYNC_SERVER` (DurableObjectNamespace):** Used to get or create instances of the `SyncServer` Durable Object. Each shared session typically maps to one `SyncServer` instance, identified by a short name derived from the `sessionID`.
*   **`env.Bucket` (R2Bucket):** Used by `SyncServer` to store shared data persistently in an R2 bucket.
*   **`secret` (string within `SyncServer` storage):** A randomly generated UUID used to authorize write operations (publish, clear) to a shared session.
*   **`sessionID` (string within `SyncServer` storage):** The original full session ID associated with the Durable Object instance.

## Usage Examples

This worker is typically invoked via HTTP requests from clients (e.g., the TUI or web interface) or through WebSocket connections established by clients.

**Example: Creating a share**
A client would send a POST request to `/share_create` with a JSON body:
```json
{
  "sessionID": "some-unique-session-identifier"
}
```
The worker would respond with:
```json
{
  "secret": "a-generated-secret-uuid",
  "url": "https://opencode.ai/s/short-id"
}
```

**Example: Subscribing to updates (WebSocket)**
A client would make a GET request to `/share_poll?id=<short-id>` with an `Upgrade: websocket` header. The `SyncServer` DO would then handle the WebSocket connection.

## Dependencies and Interactions

*   **Cloudflare Workers Runtime:** The code relies on Cloudflare Workers APIs like `DurableObject`, `WebSocketPair`, `DurableObjectState`, `R2Bucket`, etc.
*   **`node:crypto` (randomUUID):** Used to generate unique secrets for shares.
*   **SST (`infra/app.ts`):** The `infra/app.ts` file configures this worker and its `SyncServer` Durable Object binding.
*   **R2 Bucket:** The `SyncServer` uses an R2 bucket (bound as `env.Bucket`) to store shared session data persistently.
*   **Clients (TUI, Web):** Clients interact with this worker via HTTP API calls and WebSockets to share, sync, and retrieve session data.

---
*Generated by OpenCode AI Agent.*
