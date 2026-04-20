# Local Development

## Prerequisites

- Node.js and npm
- Python 3
- Ghidra for firmware analysis features
- Optional: Elasticsearch if you want register enrichment locally

## Install Dependencies

```bash
npm install
```

For Python, create the environment you use for backend and MCP work.

Example on Windows:

```powershell
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

## Start Development Mode

```bash
npm run dev
```

This script attempts to:

- start the backend on `127.0.0.1:8000`
- start the MCP service on `127.0.0.1:8001`
- start Vite for the frontend

## Useful Commands

```bash
npm run build
```

```bash
pytest -q
```

## Local URL Expectations

| Service | Default URL |
| --- | --- |
| Frontend dev server | `http://127.0.0.1:5173` |
| Backend | `http://127.0.0.1:8000` |
| MCP service | `http://127.0.0.1:8001` |

## Example Backend API Call

```bash
curl http://127.0.0.1:8000/api/v1/health
```

## Example Register Search

```bash
curl "http://127.0.0.1:8000/api/v1/registers?q=USART&size=20"
```

## Common Local Issues

!!! warning "Missing Ghidra"
    If Ghidra is not configured, project creation still works, but analysis requests will fail.

!!! warning "MCP service offline"
    The MCP UI can load, but chat and tool actions will fail until the MCP FastAPI service is reachable.
