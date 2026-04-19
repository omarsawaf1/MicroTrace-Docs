# Environment and Paths

## Frontend Environment Variables

| Variable | Purpose |
| --- | --- |
| `VITE_API_BASE_URL` | backend base URL |
| `VITE_MCP_API_BASE_URL` | MCP API base URL |

## Backend Environment Variables

| Variable | Purpose |
| --- | --- |
| `MICROTRACE_STATE_DIR` | runtime state directory |
| `MICROTRACE_OUTPUT_DIR` | analysis output directory |
| `MICROTRACE_SVD_PATH` | SVD input path |
| `MICROTRACE_GHIDRA_PATH` | Ghidra override path |
| `ELASTIC_URL` | Elasticsearch endpoint |
| `MICROTRACE_CORS_ORIGINS` | allowed origins |
| `MICROTRACE_TRUSTED_HOSTS` | trusted hosts |

## MCP Environment Variables

| Variable | Purpose |
| --- | --- |
| `MCP_RUNTIME_DIR` | runtime directory for conversations, logs, and settings |
| `LLM_PROVIDER` | default provider |
| `LLM_MODEL` | default model |
| `LLM_BASE_URL` | model endpoint |
| `OPENAI_API_KEY` | OpenAI API key |
| `SYSTEM_PROMPT` | assistant prompt override |

## Important Paths

### Local desktop mode

- Ghidra path is a real host path selected through the UI

### Docker mode

- host Ghidra path comes from `.env`
- container Ghidra path is `/opt/ghidra/support/analyzeHeadless`

## Runtime Data Locations

| Area | Location |
| --- | --- |
| backend state | `backend/runtime/state` or container volume |
| backend outputs | `backend/runtime/output` or container volume |
| MCP runtime | `mcpServer/runtime` or container volume |
