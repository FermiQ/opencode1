# `packages/web/src/pages/s/[id].astro`

## Overview

This Astro file defines a dynamic page for displaying shared OpenCode sessions. The page path is `/s/[id]`, where `[id]` is the short identifier of the shared session. It fetches session data (information and messages) from the backend API, renders the content using a Starlight layout and a custom `Share` SolidJS component, and configures OpenGraph (og) and Twitter card metadata for social sharing.

## Key Components

*   **Astro Frontmatter (--- ... ---):**
    *   **Imports:**
        *   `Base64` from `js-base64`: For encoding the title for the social card URL.
        *   `config` from `virtual:starlight/user-config`: Starlight user configuration (though not directly used in this snippet, it's a common import in Starlight pages).
        *   `StarlightPage` from `@astrojs/starlight/components/StarlightPage.astro`: The main layout component from the Starlight documentation theme.
        *   `Share` from `../../components/Share.tsx`: The SolidJS component responsible for rendering the actual shared session content.
    *   **Environment Variables:**
        *   `apiUrl = import.meta.env.VITE_API_URL`: Retrieves the backend API URL, which is set during the build process (likely by SST).
    *   **Data Fetching:**
        *   `const { id } = Astro.params;`: Gets the dynamic `id` parameter from the URL.
        *   `fetch(\`\${apiUrl}/share_data?id=\${id}\`)`: Makes a GET request to the backend's `/share_data` endpoint to retrieve the data for the specified share ID.
        *   `const data = await res.json();`: Parses the JSON response.
    *   **Error Handling:**
        *   If `!data.info` (meaning the session was not found or has no info), it returns a `404 Not Found` response.
    *   **Metadata Preparation:**
        *   `models`: Extracts a set of unique model IDs used in the assistant messages.
        *   `version`: Determines the version from `data.info.version`.
        *   `encodedTitle`: Encodes the session title for use in the OpenGraph image URL. This involves Base64 encoding and URI component encoding, plus truncation.
        *   `ogImage`: Constructs the URL for the dynamic OpenGraph social sharing card image, hosted at `social-cards.sst.dev`. This URL includes the encoded title, model IDs, version, and share ID.
*   **Astro Template (HTML-like structure):**
    *   **`<StarlightPage>` Component:**
        *   `hasSidebar={false}`: Disables the sidebar typically present in Starlight pages.
        *   `frontmatter`: Configures various aspects of the Starlight page:
            *   `title`: Sets the page title to `data.info.title`.
            *   `pagefind: false`: Disables indexing by Pagefind (if used).
            *   `template: "splash"`: Uses a specific Starlight template, likely for a full-width, focused content display.
            *   `tableOfContents: false`: Disables the table of contents.
            *   `head`: An array to inject custom tags into the `<head>` of the HTML page:
                *   Meta description tag.
                *   Meta `og:image` tag with the generated `ogImage` URL.
                *   Meta `twitter:image` tag, also with `ogImage`.
    *   **`<Share>` Component:**
        *   This is the SolidJS component that renders the main content of the shared session.
        *   `id={id}`: Passes the share ID.
        *   `api={apiUrl}`: Passes the API URL for potential client-side interactions (e.g., WebSocket).
        *   `info={data.info}`: Passes the fetched session information.
        *   `messages={data.messages}`: Passes the fetched session messages.
        *   `client:only="solid"`: An Astro directive indicating this component should only be rendered on the client-side using SolidJS.
*   **`<style is:global>`:**
    *   Contains global CSS overrides to adjust the Starlight page layout, primarily to remove padding, borders, and hide the default first content panel, allowing the `Share` component to take up the full main content area.

## Important Variables/Constants

*   **`apiUrl` (string):** The base URL for the backend API. Essential for fetching session data and potentially for the `Share` component's client-side logic.
*   **`id` (string):** The dynamic part of the URL, representing the unique short ID of the shared session.
*   **`data` (object):** The fetched JSON data from the `/share_data` endpoint, containing `info` (session metadata) and `messages` (session content).
*   **`ogImage` (string):** The dynamically generated URL for the social media preview image.

## Usage Examples

This page is accessed by navigating to a URL like `https://<your-domain>/s/<share-id>`, where `<share-id>` is a valid short identifier for a shared OpenCode session. For example: `https://opencode.ai/s/abcdef12`.

The page dynamically fetches and renders the content of that specific shared session.

## Dependencies and Interactions

*   **Astro:** The static site generator/framework used to build this page.
*   **`@astrojs/starlight`:** The documentation theme providing the `StarlightPage` layout and styling.
*   **SolidJS:** The `Share` component is a SolidJS component, indicated by `client:only="solid"` and the `.tsx` extension.
*   **`js-base64`:** Used for Base64 encoding the title for the social card URL.
*   **OpenCode Backend API (`/share_data` endpoint):** This page heavily relies on the backend API (specifically the `/share_data` endpoint defined in `packages/function/src/api.ts`) to fetch the session content. The API URL is provided via the `VITE_API_URL` environment variable.
*   **`social-cards.sst.dev`:** An external service used to generate dynamic social media preview images.
*   **`../../components/Share.tsx`:** The SolidJS component that handles the actual rendering of the session's messages and information within the page. It likely also handles any client-side interactivity, such as WebSocket connections for live updates if the session is still active.

---
*Generated by OpenCode AI Agent.*
