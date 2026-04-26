# MCP Server

## Purpose

The MCP subsystem provides AI-assisted tooling on top of local PDFs, extracted register data, and other analysis workflows.

## Internal Components

| File | Responsibility |
| --- | --- |
| `mcpServer/app.py` | FastAPI API surface for chat and config |
| `mcpServer/clientServerMcp.py` | LLM orchestration and MCP client logic |
| `mcpServer/server_mcp.py` | MCP tool server implementation |
| `mcpServer/customToolLoader.py` | trusted local custom tool loader |
| `mcpServer/*.py` tools | PDF extraction, register search, images, titles |

## Tool Exposure

The MCP server exposes tools such as:

- `extract_registers`
- `search_register`
- `online_search`
- `list_custom_tools`
- `extract_pdf_images`
- `get_pdf_titles`
- `extract_pdf_pages`
- custom tools from `mcpServer/custom_tools/*.py`

## Runtime Files

The MCP layer now keeps runtime data under `mcpServer/runtime/`, including:

- `llm_settings.json`
- `logs/`
- `conversations/`

`llm_settings.json` is created automatically on first MCP startup if it does not exist. Legacy local files such as `mcpServer/llm_settings.json` are ignored by git and are no longer read because they may contain per-user model endpoints or API keys.

## LLM Configuration

Supported providers:

- OpenAI
- Gemini
- LM Studio
- Ollama

Example model discovery behavior:

- OpenAI, Gemini, and LM Studio use `/models`
- Ollama uses `/api/tags`

Gemini uses Google's OpenAI-compatible Chat Completions endpoint:

```text
https://generativelanguage.googleapis.com/v1beta/openai/
```

## Online Search

The MCP server includes an `online_search` tool for live web lookups. It works out of the box with a no-key DuckDuckGo HTML fallback. For better reliability and structured results, configure one of these API keys:

- `BRAVE_SEARCH_API_KEY`
- `TAVILY_API_KEY`

Provider selection is controlled by `MCP_ONLINE_SEARCH_PROVIDER`:

- `auto` chooses Brave first, then Tavily, then DuckDuckGo fallback
- `brave` forces Brave Search API
- `tavily` forces Tavily Search API
- `duckduckgo` forces the no-key fallback

Optional tuning:

- `MCP_ONLINE_SEARCH_TIMEOUT` defaults to `10` seconds
- `MCP_ONLINE_SEARCH_SAFESEARCH` controls Brave safe search, defaulting to `moderate`
- `MCP_TAVILY_SEARCH_DEPTH` defaults to `basic`

The MCP composer has two separate research sends:

- **Online Search** explicitly asks the assistant to use `online_search`.
- **Deep Research** asks for a more structured firmware/project analysis. It can still use `online_search` when current web context is genuinely useful, but it should prefer local conversation, project context, and non-web tools first.

## Custom Tools

MicroTrace can load trusted local tools from:

```text
mcpServer/custom_tools/
```

Files must use this contract:

```python
TOOL_NAME = "my_custom_tool"
TOOL_DESCRIPTION = "Describe what this tool does."
TOOL_INPUT_SCHEMA = {
    "type": "object",
    "properties": {
        "query": {"type": "string"}
    },
    "required": ["query"],
}

def run(query: str) -> str:
    return f"Result for {query}"
```

Async `run()` functions are also supported.

After adding or editing a tool, press **Refresh MCP tools** in the MCP UI. The loader scans the folder each time the tool list is requested.

The MCP UI uses the loaded tool list to autocomplete working custom tool names in **Run custom tool**. Invalid tool files and built-in tools are not suggested.

!!! warning
    Custom tools are trusted local Python code and run inside the MCP tool server process. Do not add untrusted code.

## Example Query Flow

```python
messages = await client.continue_query(
    request.query,
    max_messages_context=request.max_messages_context,
)
```

## Operational Note

!!! tip
    The MCP FastAPI API is separate from the MCP tool server. The API owns HTTP endpoints and UI communication, while the tool server owns the actual tool definitions and executions.
