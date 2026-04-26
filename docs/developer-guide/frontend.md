# Frontend and Electron

## Frontend Responsibilities

The frontend provides:

- landing and informational pages
- upload and project management screens
- graph visualization
- MCP chat interface
- settings screen

## Key Entrypoints

| File | Purpose |
| --- | --- |
| `src/upload.ts` | project upload and analysis actions |
| `src/functionGraph.ts` | graph page orchestration |
| `src/settings.ts` | Ghidra settings UI |
| `src/mcpUI.ts` | MCP page bootstrap |
| `src/mcp/client.ts` | MCP UI behavior |
| `src/api.ts` | backend API helper |

The decompiled code panel includes an icon-only top-right copy action for the selected function's C code. The action is rendered with the code viewer so it stays beside the decompiled code instead of the graph toolbar.

The MCP composer exposes separate sends for normal chat, Online Search, and Deep Research. Online Search explicitly routes the request through the `online_search` tool. Deep Research asks for structured firmware/project analysis and can use online search only when current web context is useful.

The custom tool runner uses the MCP `/tools` response to autocomplete detected working custom tool names while keeping the field editable for exact manual `TOOL_NAME` entry.

## Electron Responsibilities

Electron:

- creates the desktop window
- starts backend and MCP services when needed
- exposes local APIs through preload and IPC
- reads local settings

## Startup Flow

```mermaid
flowchart TD
    A[Electron app ready] --> B[register IPC handlers]
    B --> C[read settings]
    C --> D[start backend]
    D --> E[start MCP server]
    E --> F[create main window]
```

## Example API Helper

```typescript
export const API_V1_URL = `${rawBaseUrl}/api/v1`;
```

This pattern lets the same frontend work:

- in local development
- in Electron
- behind Docker reverse proxy

## MCP Frontend Base URL

The MCP UI reads:

```typescript
import.meta.env.VITE_MCP_API_BASE_URL
```

This was important for Docker because the browser should call `/mcp` instead of a hardcoded local port.
