# Docker and Operations

## Deployment Goals

The Docker stack is designed to:

- keep frontend, backend, MCP, and Elasticsearch isolated
- expose one browser entrypoint
- support host-mounted Ghidra
- preserve runtime data in named volumes

## Compose Services

| Service | Role |
| --- | --- |
| `frontend` | nginx-based web entrypoint and reverse proxy |
| `backend` | FastAPI application |
| `mcpserver` | MCP FastAPI service |
| `elasticsearch` | register search backend |

## Reverse Proxy Behavior

```nginx
location /api/ {
    proxy_pass http://backend:8000/api/;
}

location /mcp/ {
    proxy_pass http://mcpserver:8001/;
}
```

## Volume Strategy

| Volume | Purpose |
| --- | --- |
| `elasticsearch_data` | Elasticsearch data |
| `backend_state` | backend runtime state |
| `backend_output` | analysis outputs |
| `mcp_runtime` | MCP logs, settings, and conversations |

## Security Choices

- unprivileged nginx runtime
- non-root Python containers
- `no-new-privileges:true`
- slim and Wolfi-based images where practical

## Recommended Operational Checks

1. verify `.env` contains `GHIDRA_HOST_PATH`
2. verify Ghidra mount exists on the host
3. verify backend health after boot
4. verify MCP health after boot
5. verify frontend can call `/api` and `/mcp`
