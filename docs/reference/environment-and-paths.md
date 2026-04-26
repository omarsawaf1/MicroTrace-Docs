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
| `GEMINI_API_KEY` | Gemini API key |
| `LLM_API_KEY` | optional API key fallback for local/OpenAI-compatible providers |
| `SYSTEM_PROMPT` | assistant prompt override |
| `BRAVE_SEARCH_API_KEY` | enables API-backed MCP online search through Brave |
| `TAVILY_API_KEY` | enables API-backed MCP online search through Tavily |
| `MCP_ONLINE_SEARCH_PROVIDER` | `auto`, `brave`, `tavily`, or `duckduckgo` |
| `MCP_ONLINE_SEARCH_TIMEOUT` | online search HTTP timeout in seconds |
| `MCP_ONLINE_SEARCH_SAFESEARCH` | Brave safe search level |
| `MCP_TAVILY_SEARCH_DEPTH` | Tavily search depth |

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

Per-user JSON files such as `settings.json`, `llm.json`, and MCP LLM settings are intentionally gitignored. The backend creates `settings.json` on first use, and the MCP service creates `mcpServer/runtime/llm_settings.json` on first startup.
