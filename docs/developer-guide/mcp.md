# MCP Server

## Purpose

The MCP subsystem provides AI-assisted tooling on top of local PDFs, extracted register data, and other analysis workflows.

## Internal Components

| File | Responsibility |
| --- | --- |
| `mcpServer/app.py` | FastAPI API surface for chat and config |
| `mcpServer/clientServerMcp.py` | LLM orchestration and MCP client logic |
| `mcpServer/server_mcp.py` | MCP tool server implementation |
| `mcpServer/*.py` tools | PDF extraction, register search, images, titles |

## Tool Exposure

The MCP server exposes tools such as:

- `extract_registers`
- `search_register`
- `extract_pdf_images`
- `get_pdf_titles`
- `extract_pdf_pages`

## Runtime Files

The MCP layer now keeps runtime data under `mcpServer/runtime/`, including:

- `llm_settings.json`
- `logs/`
- `conversations/`

## LLM Configuration

Supported providers:

- OpenAI
- LM Studio
- Ollama

Example model discovery behavior:

- OpenAI and LM Studio use `/models`
- Ollama uses `/api/tags`

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
