# Model Context Protocol (MCP) Server

MicroTrace integrates the **Model Context Protocol (MCP)** to securely expose extensive embedded reverse-engineering algorithms and toolsets to advanced Large Language Models and custom automation clients.

## Theoretical Overview

The Model Context Protocol establishes a standardized, bidirectional architecture that abstracts operational functionality. By deploying an MCP Server, MicroTrace provides AI orchestration models with programmatic access to distinct reverse-engineering actions, drastically accelerating autonomous file analysis capabilities. This eliminates the necessity of traditional RAG (Retrieval-Augmented Generation) pipelines by instead empowering the model with active task execution rights.

The operational workflow adheres to the following sequence:
1. **Tool Discovery:** Upon establishing an I/O connection, the MCP server emits a strict JSON Schema detailing all exposed tool functionalities and prerequisites.
2. **Context Execution:** An AI Assistant interprets the necessity of analyzing a specific binary signature and delegates the task backward over the protocol utilizing structured parameters.
3. **Internal Processing & Response:** The Python backend executes the proprietary tool logic and serializes the result into a standardized `.TextContent` packet, returning it upstream to enrich the Assistant's ongoing logical evaluation.

---

## Infrastructure Implementation

The local MicroTrace MCP ecosystem is constructed utilizing the officially maintained `mcp` SDK, and package dependencies are strictly governed by the `uv` virtual environment manager.

### System Directory
The foundational structure for the MCP service is isolated as follows:

```text
embedded-re-mcp/
├── pyproject.toml         # Canonical dependency manifest
├── src/
│   ├── server.py          # Primary Stdio-bound MCP server logic
│   ├── client.py          # Optional implementation for programmatic invocations
│   └── tools/             # Actionable analysis modules
│       ├── arch_detector.py
│       ├── disassembler.py
│       └── firmware_analyzer.py
```

### Diagnostic Tooling
The server's current architectural phase is designed to surface critical reversing integrations directly to connected models:
- **Architecture Detector (`arch_detector.py`)**: Investigates raw firmware entropy streams to determine primary instruction set architectures (ARM/MIPS/RISC-V).
- **Instruction Disassembly (`disassembler.py`)**: Extrapolates localized byte arrays into human-readable assembly instructions leveraging the Capstone framework.
- **Firmware Extrapolation (`firmware_analyzer.py`)**: Evaluates memory layout specifications and executes binwalk signature validations.

---

## Deployment Parameters

### Dependency Resolution
Virtual environment initialization must be facilitated natively by `uv` to maintain strict dependency chains.

```bash
# Initialize the local virtual context and synchronize definitions
uv init
uv sync

# Instantiate supplementary module constraints if necessary
uv add PyPDF2 mcp
```

### Server Invocation
The server communicates fundamentally utilizing standard input/output (`stdio`) streams. This allows it to be invoked as an agnostic subprocess by host environments.

```bash
# Direct subprocess invocation of the MCP Server
uv run python src/server.py
```

!!! tip "Enterprise Client Integration"
    While the protocol supports programmatic API interactions (detailed inside `src/client.py`), it integrates frictionlessly into desktop environments such as **Claude Desktop**. Integration strictly involves configuring the target application's definition profile to mirror the localized CLI invocation route.