# MCP Server Documentation


## What is MCP?

**MCP (Model Context Protocol)** enables AI assistants (like Claude) to access external tools and data.

**Analogy:**
Think of Claude as a person sitting at a desk:

- **Without MCP:** Can only answer questions from memory
- **With MCP:** Can pick up tools from the desk (calculator, file reader, analyzer) and use them

**Pattern:**

```
User asks a question
        ↓
Claude decides it needs a tool
        ↓
Claude calls your MCP server
        ↓
Server executes the tool
        ↓
Result returned to Claude
        ↓
Claude uses result to answer
```

**Example:**

```
User: "What's in firmware.bin?"
Claude: [Calls MCP tool "analyze_firmware"]
Server: [Analyzes file, returns "ARM Cortex-M, 256KB"]
Claude: "This is ARM Cortex-M firmware, 256KB in size."
```

---

## How MCP Works

### Three Main Components

1. **MCP Server**

   - Defines available tools
   - Implements tool logic
   - Runs as a separate process

```python
from mcp.server import Server
import mcp.server.stdio

app = Server("my-server")

@app.list_tools()
async def list_tools():
    return [...]

@app.call_tool()
async def call_tool(name, arguments):
    return [...]
```

2. **MCP Protocol**

   - Standard communication between Claude and your server
   - Uses stdin/stdout with JSON messages

```
Claude → {"method": "call_tool", "params": {...}} → Server
Server → {"result": "..."} → Claude
```

3. **MCP Client**

   - Connects to the server
   - Sends requests and receives responses

**Option A: Claude Desktop** (easiest)

```json
{
  "mcpServers": {
    "my-tools": {
      "command": "python",
      "args": ["server.py"]
    }
  }
}
```

**Option B: Custom Client**

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

server_params = StdioServerParameters(command="python", args=["server.py"])

async with stdio_client(server_params) as (read, write):
    async with ClientSession(read, write) as session:
        result = await session.call_tool("tool_name", {...})
```

---

## What We Did

- Learned what MCP is and how it works
- Understood that **Claude Desktop is optional**
- Created the **first MCP server** with a "hello world" tool
- Built a practical example (PDF reader)
- Planned embedded reverse-engineering tools:

  - Firmware architecture detection
  - String extractor
  - Disassembler
  - Firmware extractor
  - Memory layout analyzer
  - Peripheral database

---

## Setup Steps

1. **Install UV (Python Package Manager)**

**macOS/Linux:**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows:**

```powershell
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Verify installation:

```bash
uv --version
```

2. **Create Project**

```bash
mkdir embedded-re-mcp
cd embedded-re-mcp
uv init
```

3. **Add MCP Dependency**

```bash
uv add mcp
```

4. **Create MCP Server** (`src/server.py`)

```python
from mcp.server import Server
from mcp.types import Tool, TextContent
import mcp.server.stdio

app = Server("my-first-server")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [Tool(name="hello", description="Say hello", inputSchema={"type":"object","properties":{"name":{"type":"string"}},"required":["name"]})]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "hello":
        return [TextContent(type="text", text=f"Hello, {arguments['name']}!")]
    return [TextContent(type="text", text="Unknown tool")]

if __name__ == "__main__":
    mcp.server.stdio.stdio_server()(app)
```

5. **Run Server**

```bash
uv run python src/server.py
```

6. **Connect to Claude Desktop (Optional)**
   Edit `claude_desktop_config.json` with your server path.

7. **Custom Client Example** (`src/client.py`)

```python
import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def main():
    server_params = StdioServerParameters(command="uv", args=["run","python","src/server.py"])
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()
            result = await session.call_tool("hello", {"name": "World"})
            print(result[0].text)

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Example: PDF Reader Tool

- Install dependency:

```bash
uv add PyPDF2
```

- Create `src/pdf_server.py` with tools `read_pdf_titles` and `count_pages`
- Implements MCP pattern like the hello example

---

## Project Structure

```
embedded-re-mcp/
├── .python-version
├── pyproject.toml
├── uv.lock
├── README.md
└── src/
    ├── server.py
    ├── client.py
    ├── pdf_server.py
    └── tools/
        ├── firmware_analyzer.py
        ├── disassembler.py
        └── arch_detector.py
```

---

## Commands Reference

```bash
uv init
uv add mcp
uv add PyPDF2
uv run python src/server.py
uv run python src/client.py
uv sync
uv add package-name
```

---

## Future Work Hints

Build embedded RE tools following the MCP pattern:

1. Architecture detector (ARM/MIPS/RISC-V)
2. String extractor
3. Disassembler (Capstone)
4. Firmware extractor (Binwalk)
5. Memory layout analyzer
6. Peripheral database

---

## Key Takeaways

1. MCP is a protocol, not tied to Claude Desktop
2. Three components: Server, Protocol, Client
3. Start simple (hello or PDF reader)
4. Every tool follows the same pattern: name, description, schema, implementation
5. Use MCP anywhere: CLI, web, GUI, plugins

---

## Summary

**MCP = AI access to your tools**

**Pattern:**

1. Define tools (name, description, schema)
2. Implement tools (code)
3. Run server
4. Use from Claude or your own client

We built:

- Understanding of MCP
- UV setup and project structure
- Hello world tool
- PDF reader tool
- Plan for embedded reverse-engineering tools
  Everything else builds on this foundation.
