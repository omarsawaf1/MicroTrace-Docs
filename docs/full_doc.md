# MicroTrace - Complete Project Documentation

---

## Project Overview

**MicroTrace** is an Electron-based desktop application for automated firmware reverse engineering and SBOM (Software Bill of Materials) extraction from embedded binaries.

### Purpose

- Automatically analyze firmware binaries (.elf, .bin, .axf files)
- Extract and visualize function call graphs
- Classify software layers (MCAL, HAL, Application, Library)
- Detect vulnerabilities through CVE database integration
- Generate interactive visualizations of firmware architecture
- Provide code analysis through assembly and decompiled C code viewing

### Key Capabilities

1. **Automated Disassembly** - Process firmware using Ghidra Headless
2. **Component Classification** - YARA-based detection of MCAL/HAL functions
3. **Interactive Visualization** - D3.js-powered node-link diagrams
4. **Dashboard Analytics** - Chart.js-based statistical insights
5. **Code Inspection** - Assembly and C code viewers with syntax highlighting
6. **Vulnerability Detection** - CVE database cross-reference
7. **Project Management** - Multi-project support with localStorage persistence

---

## Architecture & Technology Stack

### Application Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Electron Main Process              │
│  (main.ts - Process Management & IPC Orchestration) │
└─────────────┬───────────────────────────────────────┘
              │
    ┌─────────┴─────────┐
    │                   │
    ▼                   ▼
┌─────────────┐   ┌──────────────┐
│  Renderer   │   │    Python    │
│  Processes  │◄──┤   Backend    │
│ (TypeScript)│   │  (engine.py) │
└─────────────┘   └──────────────┘
```

### Technology Stack

| Layer                 | Technologies                       | Purpose                              |
| --------------------- | ---------------------------------- | ------------------------------------ |
| **Desktop Framework** | Electron 39.2.3                    | Cross-platform desktop app framework |
| **Frontend Language** | TypeScript 5.9.3                   | Type-safe JavaScript                 |
| **Backend Language**  | Python 3.x                         | Firmware analysis & processing       |
| **Visualization**     | D3.js 7.9.0                        | Interactive function graphs          |
| **Charts**            | Chart.js 4.4.0                     | Dashboard analytics                  |
| **Data Storage**      | SQLite 3.x (better-sqlite3)        | Application data & analysis results  |
| **Configuration**     | OS-native config files             | User settings & preferences          |
| **IPC**               | Electron IPC (ipcMain/ipcRenderer) | Inter-process communication          |
| **Styling**           | CSS3 with CSS Variables            | Theming support                      |
| **Build Tools**       | TypeScript Compiler                | Separate configs for main/renderer   |

### Build Configuration

**Two TypeScript Configurations:**

- `tsconfig.main.json` - Main process (Node.js environment)
- `tsconfig.renderer.json` - Renderer process (Browser environment)

**Output:** All compiled JavaScript goes to `dist/` directory

---

## Project Structure

```
Grad-project/
│
├── src/                          # TypeScript source files
│   ├── main.ts                   # Electron main process entry point
│   ├── preload.ts                # Preload script for security
│   ├── renderer.ts               # Main renderer initialization
│   ├── theme.ts                  # Theme management (dark/light mode)
│   ├── dashboard.ts              # Dashboard page logic
│   ├── upload.ts                 # File upload & project management
│   ├── functionGraph.ts          # D3.js function graph visualization
│   ├── analyzerTabs.ts           # Code viewer tabs (Assembly/C)
│   ├── settings.ts               # Settings page logic
│   └── types/                    # TypeScript type definitions
│
├── frontend/                     # HTML, CSS, and static assets
│   ├── index.html                # Home page
│   ├── html/
│   │   ├── upload.html           # Upload & project management page
│   │   ├── functionGraph.html    # Function graph viewer
│   │   ├── dashboard.html        # Analytics dashboard
│   │   └── about.html            # About page
│   ├── css/
│   │   ├── main.css              # Global styles & theme variables
│   │   ├── dashboard.css         # Dashboard-specific styles
│   │   ├── upload.css            # Upload page styles
│   │   ├── functionGraph.css     # Graph visualization styles
│   │   └── analyzer-tabs.css     # Code viewer styles
│   └── assets/
│       ├── functionData.json     # Sample function graph data
│       ├── functionData2.json    # Extended function data
│       ├── asm.txt               # Assembly code dump
│       └── pictures/             # Logos and images
│
├── python/                       # Python backend
│   ├── engine.py                 # Main Python engine (stdin/stdout communication)
│   ├── requirements.txt          # Python dependencies
│   ├── core/                     # Core functionality
│   │   ├── __init__.py
│   │   ├── output.py             # Safe output handling
│   │   └── async_processor.py   # Async command processing
│   └── handlers/                 # Command handlers
│       ├── __init__.py
│       ├── basic.py              # Basic command handlers
│       ├── api.py                # API integration handlers
│       ├── file.py               # File operations
│       ├── math.py               # Mathematical operations
│       ├── string.py             # String manipulation
│       ├── list.py               # List operations
│       ├── dict.py               # Dictionary operations
│       ├── datetime.py           # Date/time operations
│       └── exec.py               # Code execution (security risk!)
│
├── dist/                         # Compiled JavaScript (generated)
│   ├── main.js
│   ├── renderer.js
│   ├── dashboard.js
│   └── ...
│
├── node_modules/                 # NPM dependencies
├── package.json                  # Node.js project configuration
├── tsconfig.main.json            # TypeScript config for main process
├── tsconfig.renderer.json        # TypeScript config for renderer
├── .gitignore
└── README.md

```

---

## Core Features & Functionality

### 1. **File Upload & Project Management** (`upload.ts`)

**Purpose:** Manage multiple firmware analysis projects

**Features:**

- File upload with validation (.bin, .elf, .axf)
- Project metadata storage (name, path, CPU family, flash address)
- Active project selection
- localStorage-based persistence
- Project deletion

**Data Model:**

```typescript
interface Project {
  id: string; // Timestamp-based unique ID
  name: string; // User-defined or filename
  path: string; // Absolute file path (via Electron API)
  flashAddress: string; // Flash memory address (e.g., 0x08000000)
  cpuFamily: string; // CPU architecture (ARM Cortex-M0, M3, M4, etc.)
  dateAdded: number; // Timestamp
}
```

**Key Functions:**

- `initializeUpload()` - Initialize page and load projects from DB
- `loadProjects()` - Query projects table via IPC
- `saveProject(project)` - Insert/update in SQLite
- `setActiveProject(id)` - Update active_project table
- `deleteProject(id)` - Remove from database (cascade)
- `renderProjects()` - Update UI list

**Database Storage:**

- All projects stored in SQLite `projects` table
- Active project in `active_project` table
- See [SQLite Database Integration](#10-sqlite-database-integration) section for schema

---

### 2. **Function Graph Visualization** (`functionGraph.ts`)

**Purpose:** Interactive visualization of firmware function call relationships

**Technology:** D3.js force-directed graph

**Features:**

- Node-link diagram with force simulation
- Color-coded by function type (MCAL, HAL, Other)
- Interactive pan & zoom
- Drag-and-drop node positioning
- Hover effects with neighbor highlighting
- Click to view function details
- Search functionality
- Type filtering (legend with checkboxes)
- Depth filtering

**Node Data Structure:**

```typescript
interface NodeData {
  id: string; // Function identifier (e.g., "FUN_08000580")
  name: string; // Function name
  type: "mcal" | "hal" | "other" | "other2";
  info?: string; // Additional description
  depth?: number; // Call depth from root
  address?: string; // Memory address (hex)
  instruction_count?: number;
  asm_start_line?: number; // Line in asm.txt
  c_code?: string; // Decompiled C code
  data_calls?: string[]; // Data references
  calls?: string[]; // Function calls
}

interface LinkData {
  source: string; // Source node ID
  target: string; // Target node ID
}
```

**Key Functions:**

- `loadData()` - Fetch JSON data (functionData2.json)
- `mountGraph(data)` - Create D3 visualization
- `createLegend()` - Build filter UI
- `showPanelForNode(id)` - Display side panel
- `wireSearch(data)` - Connect search input
- `applyFilters()` - Filter by type/depth

**Interaction Features:**

- **Pan/Zoom:** Mouse wheel, drag background
- **Node Drag:** Constrain to canvas bounds
- **Hover:** Highlight connected nodes/links
- **Click:** Show details panel + center node
- **Arrows:** Directional markers on links

**Color Scheme (CSS Variables):**

```css
--mcal: #1f77b4 (blue)
--hal: #ff7f0e (orange)
--other: #6baed6 (light blue)
```

---

### 3. **Analyzer Tabs** (`analyzerTabs.ts`)

**Purpose:** Display assembly and decompiled C code for selected functions

**Features:**

- Tabbed interface (Assembly / C Code)
- Syntax highlighting (custom regex-based)
- Assembly file caching
- Function metadata display (address, depth, instruction count)

**Tab Structure:**

- **Assembly Tab:** Raw ARM assembly from asm.txt
- **C Code Tab:** Ghidra-decompiled C code

**Key Functions:**

- `initializeAnalyzerTabs()` - Set up tab click handlers
- `loadAssemblyFile()` - Fetch and cache asm.txt
- `getAssemblyForFunction(startLine, count)` - Extract lines
- `applySyntaxHighlight(text, language)` - Colorize code
- `updateAssemblyTab(nodeData)` - Populate assembly view
- `updateCCodeTab(nodeData)` - Populate C view
- `handleNodeClickForTabs(nodeData)` - Called from graph

**Syntax Highlighting:**

**Assembly (`asm` mode):**

- Comments: `;...`
- Keywords: mov, ldr, str, add, sub, push, pop, etc.
- Hex numbers: `0x[0-9a-f]+`
- Registers: `r0-r15`

**C Code (`c` mode):**

- Comments: `//...` and `/* ... */`
- Keywords: if, else, return, int, void, undefined, param*, DAT*, FUN\_
- Strings: `"..."`, `'...'`
- Hex numbers: `0x[0-9a-f]+`

---

### 4. **Dashboard Analytics** (`dashboard.ts`)

**Purpose:** Statistical visualization of firmware analysis results

**Technology:** Chart.js 4.x

**Charts:**

1. **Layer Distribution** (Doughnut) - MCAL/HAL/Other/Library breakdown
2. **Library Usage** (Bar) - Which libraries are used
3. **Vulnerability Distribution** (Pie) - Severity levels
4. **Component Count** (Line) - Functions/Libraries/Drivers/Middleware

**Data Model:**

```typescript
interface FunctionData {
  name: string;
  type: "mcal" | "hal" | "other" | "library";
  version?: string;
  cve?: string; // CVE identifier
  severity?: "none" | "low" | "medium" | "high" | "critical";
}

interface AnalysisData {
  functions: FunctionData[];
  layerDistribution: { [key: string]: number };
  libraryUsage: { [key: string]: number };
  vulnerabilities: { [key: string]: number };
  components: { [key: string]: number };
}
```

**Key Functions:**

- `initializeDashboard()` - Setup and listen for Python output
- `analyzeFirmware(project)` - Send analysis request
- `populateTable(functions)` - Fill table rows
- `createCharts(data)` - Initialize Chart.js instances
- Theme observer - Update chart colors on theme change

**Workflow:**

1. User clicks "Analyze" button
2. Send project path to Python via IPC
3. Python processes firmware (Ghidra, YARA, etc.)
4. Python sends JSON results back
5. Parse JSON and update charts/table

---

### 5. **Theme Management** (`theme.ts`)

**Purpose:** Dark/Light mode switching with persistence

**Implementation:**

- CSS variables for all colors
- `data-light-theme` attribute on `<body>`
- localStorage persistence
- Sidebar toggle button

**CSS Variables (subset):**

```css
/* Dark Mode */
--bg-primary: #0b1220
--bg-secondary: #162032
--text-primary: #e5e7eb
--accent: #667eea

/* Light Mode */
--bg-primary: #f9fafb
--bg-secondary: #ffffff
--text-primary: #1a1a2e
--accent: #4f46e5
```

**Functions:**

- `getTheme()` - Read from localStorage
- `setTheme(isDark)` - Apply and persist
- `toggleTheme()` - Switch modes
- Observers in graphs/charts update on theme change

---

### 6. **Settings Page** (`settings.ts`)

**Purpose:** Application configuration

**Settings:**

- Ghidra installation path (for future automation)
- Theme preference
- Other tool paths (planned)

**Storage:**

- OS-native configuration files (platform-specific)
  - **Windows:** `%APPDATA%/MicroTrace/config.json`
  - **macOS:** `~/Library/Application Support/MicroTrace/config.json`
  - **Linux:** `~/.config/MicroTrace/config.json`
- IPC handlers: `config:get`, `config:set`, `config:get-all`, `config:reset`
- See [OS-Native Configuration Files](#11-os-native-configuration-files) section for implementation

**Interface:**

```typescript
interface Settings {
  ghidraPath: string;
  theme: "dark" | "light";
}
```

---

### 7. **Electron Main Process** (`main.ts`)

**Purpose:** Application lifecycle and IPC orchestration

**Responsibilities:**

- Create browser windows
- Spawn Python subprocess
- Handle IPC communication (bi-directional)
- Manage settings store (in-memory)
- Window lifecycle management

**Python Integration:**

```javascript
// Spawn Python with unbuffered output
py = spawn("python", ["-u", path.join(__dirname, "../python/engine.py")], {
  stdio: ["pipe", "pipe", "pipe"],
});

// Forward Python stdout to renderer
py.stdout.on("data", (data) => {
  win.webContents.send("py-out", data.toString());
});

// Send commands from renderer to Python stdin
ipcMain.handle("py-in", async (_event, text: string) => {
  py.stdin.write(text.trim() + "\n");
});
```

**IPC Handlers:**

- `py-in` - Send command to Python
- `py-out` - Receive output from Python
- `settings-get` - Get setting value
- `settings-set` - Update setting
- `settings-get-all` - Get all settings

**Security:**

- `contextIsolation: true`
- `nodeIntegration: false`
- Preload script for controlled API exposure

---

### 8. **Preload Script** (`preload.ts`)

**Purpose:** Secure bridge between renderer and main process

**Exposed API:**

```typescript
window.api = {
  send: (txt: string) => ipcRenderer.invoke("py-in", txt),
  onOutput: (cb: (data: string) => void) =>
    ipcRenderer.on("py-out", (_, data) => cb(data)),
  getFilePath: (file: File) => file.path,
  settingsGet: (key: string) => ipcRenderer.invoke("settings-get", key),
  settingsSet: (key: string, value: any) =>
    ipcRenderer.invoke("settings-set", key, value),
  settingsGetAll: () => ipcRenderer.invoke("settings-get-all"),
};
```

---

### 9. **Python Backend** (`python/engine.py`)

**Purpose:** Command processor for firmware analysis

**Architecture:**

- Event loop reading from stdin
- Command routing to handlers
- Results printed to stdout (JSON)
- Unbuffered output for real-time communication

**Core (`core/`):**

- `output.py` - `safe_print()` with UTF-8 encoding
- `async_processor.py` - Async command execution

**Handlers (`handlers/`):**

- `basic.py` - Echo, help, version
- `api.py` - External API calls
- `file.py` - File operations
- `math.py` - Calculations
- `string.py` - String manipulation
- `list.py` - List operations
- `dict.py` - Dictionary operations
- `datetime.py` - Date/time
- `exec.py` - **SECURITY RISK**: Code execution

**Command Flow:**

```
1. Electron sends command via stdin
2. engine.py reads line
3. process_command() routes to handler
4. Handler executes and returns result
5. safe_print() sends JSON to stdout
6. Electron receives via py-out event
```

**Example Commands:**

```python
# Help
> help

# File analysis (to be implemented)
> analyze /path/to/firmware.elf
```

---

### 10. **Ghidra Headless Integration**

**Purpose:** Automated firmware disassembly and decompilation using Ghidra

**Files:**

- `python/headless_script.txt` - Command template for analyzeHeadless
- `python/export_disasm.py` - Jython post-script for exporting analysis results

**Workflow:**

```
1. User uploads firmware (.bin/.elf) in Upload tab
2. Frontend sends analysis request to Python via IPC
3. Python constructs Ghidra command from template
4. Ghidra analyzeHeadless processes firmware
5. export_disasm.py exports disassembly + decompiled C
6. Python parses output and stores in SQLite
7. Frontend displays results in Function Graph & Dashboard
```

**Ghidra Command Structure:**

```bash
<analyzeHeadless.bat location> "." MyProject \
  -import "<.bin or .elf file location>" \
  -loader BinaryLoader \
  -loader-baseAddr <flash memory start address> \
  -loader-blockName FLASH \
  -processor "<CPU family:Little endian or big endian:instruction/data model:architecture version>" \
  -postScript export_disasm.py <flash memory start address> 0x0 ".\asm.txt" ".\code.txt" "<.bin or .elf file location>" \
  -deleteProject
```

**Command Parameters:**

| Parameter                  | Example                                 | Description                      |
| -------------------------- | --------------------------------------- | -------------------------------- |
| `analyzeHeadless location` | `C:\ghidra\support\analyzeHeadless.bat` | Path to Ghidra's headless script |
| `project directory`        | `"."`                                   | Temporary project directory      |
| `project name`             | `MyProject`                             | Temporary project name           |
| `-import`                  | `"C:\firmware.bin"`                     | Firmware file to analyze         |
| `-loader`                  | `BinaryLoader`                          | Ghidra loader type               |
| `-loader-baseAddr`         | `0x08000000`                            | Flash memory start address       |
| `-loader-blockName`        | `FLASH`                                 | Memory block name                |
| `-processor`               | `"ARM:LE:32:v7"`                        | CPU architecture specification   |
| `-postScript`              | `export_disasm.py`                      | Script to run after analysis     |
| `-deleteProject`           | -                                       | Clean up temporary project       |

**Processor String Format:**

```
<CPU family>:<Endianness>:<Data Model>:<Architecture Version>
```

**Examples:**

- ARM Cortex-M0: `"ARM:LE:32:v6"`
- ARM Cortex-M3: `"ARM:LE:32:v7"`
- ARM Cortex-M4: `"ARM:LE:32:v7"`
- ARM Cortex-M7: `"ARM:LE:32:v7"`
- STM32 (Thumb-2): `"ARM:LE:32:Cortex"`

**export_disasm.py Script:**

**Purpose:** Jython post-script that exports Ghidra analysis to text files

**Inputs (passed as script args):**

1. `flashStartHex` - Flash memory base address (e.g., `0x08000000`)
2. `lengthHex` - Firmware length in hex (use `0x0` for auto-detect)
3. `outDisasmPath` - Output path for disassembly (e.g., `.\asm.txt`)
4. `outAsmPath` - Output path for decompiled C (e.g., `.\code.txt`)
5. `firmwarePath` - Original firmware file path

**Outputs:**

**1. Disassembly File (`asm.txt`):**
Tab-separated format with columns:

- Address (hex)
- Bytes (hex string)
- Function name (if at function entry)
- Data type (if data unit)
- Instruction/data text
- Metadata (comments, references, values)

**Example:**

```
08000000    10 B5 04 4B    FUN_08000000        push {r4,lr} ; REFS: DATA->08000004
08000002    78 44                              add r0,pc
08000004    00 F0 14 F8                        bl FUN_08000030
```

**2. Decompiled C File (`code.txt`):**
Contains decompiled C code for each function:

```c
// Function: FUN_08000000 @ 08000000
void FUN_08000000(void)
{
  init_peripherals();
  read_sensor();
  return;
}

// Function: init_peripherals @ 08000030
void init_peripherals(void)
{
  *(uint *)(DAT_08000044 + 0x40) = *(uint *)(DAT_08000044 + 0x40) | 0x10000;
  return;
}
```

**Fallback:** If decompilation fails, falls back to raw assembly instructions

**Key Features:**

- **Smart length detection:** Reads firmware file size if length=0x0
- **Memory validation:** Checks if bytes are mapped at base address
- **Comprehensive metadata:** Captures comments, references, data types
- **Error handling:** Graceful fallback to assembly if decompilation fails
- **UTF-8 encoding:** Proper handling of special characters

**Python Integration:**

```python
# python/handlers/ghidra_handler.py
import subprocess
import os
from pathlib import Path

def analyze_firmware(project_path: str, flash_addr: str, cpu_family: str, processor: str) -> dict:
    """
    Run Ghidra headless analysis on firmware

    Args:
        project_path: Path to firmware file
        flash_addr: Flash memory base address (e.g., "0x08000000")
        cpu_family: CPU family name (e.g., "ARM Cortex-M4")
        processor: Ghidra processor ID (e.g., "ARM:LE:32:v7")

    Returns:
        dict with paths to asm.txt and code.txt
    """
    # Get Ghidra path from config
    ghidra_path = config.get('ghidraPath')
    if not ghidra_path or not os.path.exists(ghidra_path):
        raise ValueError("Ghidra path not configured or invalid")

    # Setup output paths
    output_dir = Path(project_path).parent
    asm_output = output_dir / "asm.txt"
    code_output = output_dir / "code.txt"

    # Construct command
    cmd = [
        ghidra_path,  # analyzeHeadless.bat
        ".",  # temp project dir
        "MicroTraceAnalysis",  # project name
        "-import", project_path,
        "-loader", "BinaryLoader",
        "-loader-baseAddr", flash_addr,
        "-loader-blockName", "FLASH",
        "-processor", processor,
        "-postScript", "export_disasm.py",
        flash_addr, "0x0",  # auto-detect length
        str(asm_output),
        str(code_output),
        project_path,
        "-deleteProject"
    ]

    # Run Ghidra
    try:
        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True,
            timeout=300  # 5 minute timeout
        )

        if result.returncode != 0:
            raise RuntimeError(f"Ghidra analysis failed: {result.stderr}")

        # Verify outputs exist
        if not asm_output.exists() or not code_output.exists():
            raise RuntimeError("Ghidra did not produce expected output files")

        return {
            "success": True,
            "asm_path": str(asm_output),
            "code_path": str(code_output),
            "log": result.stdout
        }

    except subprocess.TimeoutExpired:
        raise RuntimeError("Ghidra analysis timed out after 5 minutes")
    except Exception as e:
        raise RuntimeError(f"Ghidra analysis error: {str(e)}")
```

**Post-Processing:**

After Ghidra analysis, Python should:

1. Parse `asm.txt` to extract function addresses and names
2. Parse `code.txt` to extract decompiled C code per function
3. Build function call graph from references
4. Store results in SQLite database
5. Send JSON to frontend for visualization

**Error Handling:**

Common issues and solutions:

- **No memory block at address:** Verify `-loader-baseAddr` matches actual flash address
- **No instructions produced:** Check processor string matches target CPU
- **Decompilation failed:** Script falls back to assembly (normal for complex code)
- **Timeout:** Increase timeout for large firmware files
- **File not found:** Verify Ghidra path in settings

---

## Data Flow & Communication

### Overall Data Flow

```
┌──────────────┐
│   User       │
└──────┬───────┘
       │ Upload firmware file
       ▼
┌──────────────────────┐
│  upload.ts           │
│  - Validate file     │
│  - Store project     │
│  - Save to localStorage
└──────────────────────┘
       │
       │ User clicks "Analyze"
       ▼
┌──────────────────────┐
│  dashboard.ts        │
│  - Get active project│
│  - Send path via IPC │
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│  main.ts (Electron)  │
│  - Receive IPC       │
│  - Forward to Python │
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│  engine.py (Python)  │
│  - Receive command   │
│  - Execute Ghidra (future)
│  - Apply YARA rules (future)
│  - Generate JSON    │
└──────┬───────────────┘
       │
       │ JSON results
       ▼
┌──────────────────────┐
│  main.ts             │
│  - Forward to renderer│
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│  dashboard.ts        │
│  - Parse JSON        │
│  - Update charts     │
│  - Populate table    │
└──────────────────────┘
       │
       │ User clicks "Function Graph"
       ▼
┌──────────────────────┐
│  functionGraph.ts    │
│  - Load functionData2.json
│  - Create D3 graph  │
└──────┬───────────────┘
       │
       │ User clicks node
       ▼
┌──────────────────────┐
│  analyzerTabs.ts     │
│  - Load asm.txt      │
│  - Show assembly/C   │
│  - Syntax highlight  │
└──────────────────────┘
```

### IPC Communication Patterns

**Pattern 1: Renderer → Main → Python**

```javascript
// Renderer (dashboard.ts)
window.api.send(project.path);

// Main (main.ts)
ipcMain.handle("py-in", (_, text) => {
  py.stdin.write(text + "\n");
});

// Python (engine.py)
line = sys.stdin.readline();
process_command(line);
```

**Pattern 2: Python → Main → Renderer**

```python
# Python (engine.py)
safe_print(json.dumps(results))

# Main (main.ts)
py.stdout.on("data", (data) => {
  win.webContents.send("py-out", data.toString())
})

# Renderer (dashboard.ts)
window.api.onOutput((data) => {
  const parsed = JSON.parse(data)
  createCharts(parsed)
})
```

---

## Security Considerations

### Current Security Issues

1. **Python Code Execution** (`handlers/exec.py`)

   - Arbitrary code execution via stdin
   - **Critical vulnerability**
   - **Recommendation:** Remove or sandbox

2. **Path Exposure** (`preload.ts`)

   - `file.path` exposes full filesystem paths
   - **Recommendation:** Use Electron's dialog API

3. **No Input Validation**

   - Python commands not sanitized
   - File uploads not thoroughly validated
   - **Recommendation:** Whitelist commands, validate file signatures

4. **localStorage Security**

   - Project data in plain text
   - Sensitive paths exposed
   - **Recommendation:** Encrypt sensitive data

5. **No Authentication**

   - Local desktop app, but if networked...
   - **Recommendation:** Add authentication if exposing APIs

6. **Electron Security**

   - ✅ Context isolation enabled
   - ✅ Node integration disabled
   - ❌ CSP (Content Security Policy) not configured

7. **Python Subprocess**
   - No validation of Python executable path
   - Could be hijacked if PATH is compromised
   - **Recommendation:** Use absolute paths

### Security Best Practices to Implement

```typescript
// 1. Validate file types by signature, not extension
function validateFirmwareFile(buffer: Buffer): boolean {
  // Check ELF magic number
  if (buffer[0] === 0x7F && buffer[1] === 0x45 &&
      buffer[2] === 0x4C && buffer[3] === 0x46) {
    return true; // ELF file
  }
  return false;
}

// 2. Whitelist Python commands
const ALLOWED_COMMANDS = ["help", "analyze", "version"];
function validateCommand(cmd: string): boolean {
  return ALLOWED_COMMANDS.includes(cmd.split(" ")[0]);
}

// 3. Use Electron dialog for file selection
const { dialog } = require("electron");
const result = await dialog.showOpenDialog({
  properties: ["openFile"],
  filters: [
    { name: "Firmware", extensions: ["elf", "bin", "axf"] }
  ]
});

// 4. Encrypt sensitive data in localStorage
import crypto from "crypto";
function encryptData(data: string, key: string): string {
  const cipher = crypto.createCipher("aes-256-cbc", key);
  return cipher.update(data, "utf8", "hex") + cipher.final("hex");
}

// 5. Implement CSP
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self'; script-src 'self'">

// 6. Sanitize Python output
function sanitizeOutput(data: string): string {
  try {
    JSON.parse(data); // Validate JSON
    return data;
  } catch {
    return JSON.stringify({ error: "Invalid output" });
  }
}
```

---

## Best Practices & Recommendations

### Architecture Improvements

#### 1. **Separate Concerns**

```
Current: Single files with mixed responsibilities
Recommended:
  src/
    services/      # Business logic
      FirmwareService.ts
      ProjectService.ts
      AnalysisService.ts
    components/    # UI components (if using framework)
      Graph/
      Dashboard/
      Upload/
    utils/         # Shared utilities
      validation.ts
      formatting.ts
    types/         # TypeScript types
```

#### 2. **State Management**

- **Current:** localStorage + in-memory
- **Recommended:** Redux/Zustand + Electron Store
  - Centralized state
  - Time-travel debugging
  - Persistent store with encryption

#### 3. **Error Handling**

```typescript
// Current: Minimal error handling
// Recommended: Comprehensive error boundaries

class AnalysisError extends Error {
  constructor(message: string, public code: string, public details?: any) {
    super(message);
  }
}

async function analyzeFirmware(path: string): Promise<Result> {
  try {
    validate(path);
    const result = await pythonService.analyze(path);
    return { success: true, data: result };
  } catch (error) {
    if (error instanceof AnalysisError) {
      logger.error(error.code, error.details);
      return { success: false, error: error.message };
    }
    throw error; // Re-throw unexpected errors
  }
}
```

#### 4. **Logging System**

```typescript
import winston from "winston";

const logger = winston.createLogger({
  level: "info",
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: "error.log", level: "error" }),
    new winston.transports.File({ filename: "combined.log" }),
  ],
});

logger.info("Analysis started", { projectId, filePath });
```

#### 5. **Testing Strategy**

```
tests/
  unit/
    services/
      FirmwareService.test.ts
      ProjectService.test.ts
    utils/
      validation.test.ts
  integration/
    python-ipc.test.ts
    file-upload.test.ts
  e2e/
    full-analysis-flow.test.ts
```

**Frameworks:**

- Jest for unit tests
- Spectron/Playwright for E2E
- Mock Electron APIs

#### 6. **Python Backend Restructuring**

```
python/
  src/
    main.py                # Entry point
    config.py              # Configuration
    models/
      firmware.py          # Data models
      analysis_result.py
    services/
      ghidra_service.py    # Ghidra integration
      yara_service.py      # YARA classification
      cve_service.py       # Vulnerability lookup
    utils/
      validators.py
      formatters.py
  tests/
    test_ghidra_service.py
    test_yara_service.py
  requirements/
    base.txt
    dev.txt
    prod.txt
```

#### 7. **API Versioning**

```typescript
// Future-proof IPC channels
enum IpcChannel {
  PYTHON_COMMAND_V1 = "py-in:v1",
  PYTHON_OUTPUT_V1 = "py-out:v1",
  SETTINGS_GET_V1 = "settings-get:v1",
}
```

#### 8. **Configuration Management**

```javascript
// config.ts
export const config = {
  python: {
    executable: process.env.PYTHON_PATH || "python",
    engineScript: "../python/engine.py",
    timeout: 60000, // ms
  },
  ghidra: {
    installPath: process.env.GHIDRA_PATH || "",
    analyzeScript: "analyzeHeadless",
  },
  security: {
    encryptionKey: process.env.ENCRYPTION_KEY,
    allowedFileTypes: [".elf", ".bin", ".axf"],
  },
  ui: {
    defaultTheme: "dark",
    maxProjects: 50,
  },
};
```

#### 9. **Type Safety**

```typescript
// Create shared types package
types / index.ts;
firmware.ts;
analysis.ts;
project.ts;

// firmware.ts
export enum FunctionType {
  MCAL = "mcal",
  HAL = "hal",
  OTHER = "other",
  LIBRARY = "library",
}

export interface FirmwareFunction {
  readonly id: string;
  readonly name: string;
  readonly type: FunctionType;
  address: string;
  depth?: number;
  instructionCount?: number;
  calls: readonly string[];
  // ... more fields
}

// Use const assertions
const COLORS = {
  mcal: "#1f77b4",
  hal: "#ff7f0e",
  other: "#6baed6",
} as const;
```

#### 10. **SQLite Database Integration**

**Why SQLite?**

- ✅ Serverless, zero-configuration
- ✅ Fast queries with indexes
- ✅ ACID transactions
- ✅ Better data integrity than JSON files
- ✅ Supports complex queries and relationships
- ✅ Perfect for Electron desktop apps

**Library:** `better-sqlite3` (synchronous, faster than node-sqlite3)

**Database Schema:**

```sql
-- Projects table
CREATE TABLE projects (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  path TEXT NOT NULL UNIQUE,
  flash_address TEXT NOT NULL,
  cpu_family TEXT NOT NULL,
  date_added INTEGER NOT NULL,
  last_modified INTEGER DEFAULT (strftime('%s', 'now'))
);

-- Active project (single row)
CREATE TABLE active_project (
  id INTEGER PRIMARY KEY CHECK (id = 1),
  project_id TEXT REFERENCES projects(id) ON DELETE SET NULL
);

-- Function graph data
CREATE TABLE function_nodes (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  project_id TEXT NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  node_id TEXT NOT NULL,
  name TEXT NOT NULL,
  type TEXT NOT NULL CHECK(type IN ('mcal', 'hal', 'other', 'library')),
  address TEXT,
  depth INTEGER,
  instruction_count INTEGER,
  c_code TEXT,
  asm_start_line INTEGER,
  UNIQUE(project_id, node_id)
);

CREATE INDEX idx_function_nodes_project ON function_nodes(project_id);
CREATE INDEX idx_function_nodes_type ON function_nodes(type);

-- Function calls (edges)
CREATE TABLE function_calls (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  project_id TEXT NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  source_id TEXT NOT NULL,
  target_id TEXT NOT NULL,
  FOREIGN KEY (project_id, source_id) REFERENCES function_nodes(project_id, node_id),
  FOREIGN KEY (project_id, target_id) REFERENCES function_nodes(project_id, node_id)
);

CREATE INDEX idx_function_calls_project ON function_calls(project_id);
CREATE INDEX idx_function_calls_source ON function_calls(source_id);

-- Assembly code cache
CREATE TABLE assembly_cache (
  project_id TEXT PRIMARY KEY REFERENCES projects(id) ON DELETE CASCADE,
  assembly_text TEXT NOT NULL,
  cached_at INTEGER DEFAULT (strftime('%s', 'now'))
);

-- Analysis results (dashboard data)
CREATE TABLE analysis_results (
  project_id TEXT PRIMARY KEY REFERENCES projects(id) ON DELETE CASCADE,
  layer_distribution TEXT,  -- JSON: {"MCAL": 15, "HAL": 28, ...}
  library_usage TEXT,        -- JSON
  vulnerabilities TEXT,      -- JSON
  components TEXT,           -- JSON
  analyzed_at INTEGER DEFAULT (strftime('%s', 'now'))
);

-- CVE data
CREATE TABLE cve_findings (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  project_id TEXT NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  function_name TEXT NOT NULL,
  cve_id TEXT NOT NULL,
  severity TEXT CHECK(severity IN ('none', 'low', 'medium', 'high', 'critical')),
  description TEXT
);

CREATE INDEX idx_cve_project ON cve_findings(project_id);
```

**Implementation:**

```typescript
// src/services/database.ts
import Database from "better-sqlite3";
import { app } from "electron";
import { join } from "path";

const dbPath = join(app.getPath("userData"), "microtrace.db");
const db = new Database(dbPath);

// Enable foreign keys
db.pragma("foreign_keys = ON");

// Initialize schema
function initializeDatabase() {
  const schema = `
    -- (paste schema above)
  `;
  db.exec(schema);
}

// Project operations
export const projectDb = {
  getAll: () => {
    return db.prepare("SELECT * FROM projects ORDER BY date_added DESC").all();
  },

  getActive: () => {
    const result = db
      .prepare(
        `
      SELECT p.* FROM projects p
      JOIN active_project ap ON p.id = ap.project_id
    `
      )
      .get();
    return result || null;
  },

  insert: (project: Project) => {
    const stmt = db.prepare(`
      INSERT INTO projects (id, name, path, flash_address, cpu_family, date_added)
      VALUES (@id, @name, @path, @flashAddress, @cpuFamily, @dateAdded)
    `);
    stmt.run(project);
  },

  setActive: (projectId: string) => {
    // Use transaction for consistency
    const setActive = db.transaction((id: string) => {
      db.prepare("DELETE FROM active_project").run();
      db.prepare(
        "INSERT INTO active_project (id, project_id) VALUES (1, ?)"
      ).run(id);
    });
    setActive(projectId);
  },

  delete: (projectId: string) => {
    db.prepare("DELETE FROM projects WHERE id = ?").run(projectId);
  },
};

// Function graph operations
export const graphDb = {
  saveNodes: (projectId: string, nodes: NodeData[]) => {
    const insertNode = db.prepare(`
      INSERT OR REPLACE INTO function_nodes 
      (project_id, node_id, name, type, address, depth, instruction_count, c_code, asm_start_line)
      VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)
    `);

    const insertMany = db.transaction((nodes: NodeData[]) => {
      for (const node of nodes) {
        insertNode.run(
          projectId,
          node.id,
          node.name,
          node.type,
          node.address,
          node.depth,
          node.instruction_count,
          node.c_code,
          node.asm_start_line
        );
      }
    });

    insertMany(nodes);
  },

  saveLinks: (projectId: string, links: LinkData[]) => {
    const insertLink = db.prepare(`
      INSERT OR REPLACE INTO function_calls (project_id, source_id, target_id)
      VALUES (?, ?, ?)
    `);

    const insertMany = db.transaction((links: LinkData[]) => {
      for (const link of links) {
        insertLink.run(projectId, link.source, link.target);
      }
    });

    insertMany(links);
  },

  getGraphData: (projectId: string): GraphData => {
    const nodes = db
      .prepare(
        `
      SELECT node_id as id, name, type, address, depth, instruction_count, c_code, asm_start_line
      FROM function_nodes
      WHERE project_id = ?
    `
      )
      .all(projectId);

    const links = db
      .prepare(
        `
      SELECT source_id as source, target_id as target
      FROM function_calls
      WHERE project_id = ?
    `
      )
      .all(projectId);

    return { nodes, links };
  },
};

// Assembly cache operations
export const assemblyDb = {
  save: (projectId: string, assemblyText: string) => {
    db.prepare(
      `
      INSERT OR REPLACE INTO assembly_cache (project_id, assembly_text)
      VALUES (?, ?)
    `
    ).run(projectId, assemblyText);
  },

  get: (projectId: string): string | null => {
    const result = db
      .prepare(
        `
      SELECT assembly_text FROM assembly_cache WHERE project_id = ?
    `
      )
      .get(projectId);
    return result?.assembly_text || null;
  },
};

// Export db instance for custom queries
export default db;
```

**IPC Handlers (main.ts):**

```typescript
import { projectDb, graphDb, assemblyDb } from "./services/database";

ipcMain.handle("db:get-projects", async () => projectDb.getAll());
ipcMain.handle("db:get-active-project", async () => projectDb.getActive());
ipcMain.handle("db:save-project", async (_, project) =>
  projectDb.insert(project)
);
ipcMain.handle("db:set-active-project", async (_, id) =>
  projectDb.setActive(id)
);
ipcMain.handle("db:delete-project", async (_, id) => projectDb.delete(id));

ipcMain.handle("db:get-graph-data", async (_, projectId) =>
  graphDb.getGraphData(projectId)
);
ipcMain.handle("db:save-graph-data", async (_, projectId, nodes, links) => {
  graphDb.saveNodes(projectId, nodes);
  graphDb.saveLinks(projectId, links);
});

ipcMain.handle("db:get-assembly", async (_, projectId) =>
  assemblyDb.get(projectId)
);
ipcMain.handle("db:save-assembly", async (_, projectId, text) =>
  assemblyDb.save(projectId, text)
);
```

**Preload API:**

```typescript
// preload.ts
contextBridge.exposeInMainWorld("api", {
  // ... existing API
  db: {
    getProjects: () => ipcRenderer.invoke("db:get-projects"),
    getActiveProject: () => ipcRenderer.invoke("db:get-active-project"),
    saveProject: (project: Project) =>
      ipcRenderer.invoke("db:save-project", project),
    setActiveProject: (id: string) =>
      ipcRenderer.invoke("db:set-active-project", id),
    deleteProject: (id: string) => ipcRenderer.invoke("db:delete-project", id),
    getGraphData: (projectId: string) =>
      ipcRenderer.invoke("db:get-graph-data", projectId),
    saveGraphData: (projectId: string, nodes: NodeData[], links: LinkData[]) =>
      ipcRenderer.invoke("db:save-graph-data", projectId, nodes, links),
    getAssembly: (projectId: string) =>
      ipcRenderer.invoke("db:get-assembly", projectId),
    saveAssembly: (projectId: string, text: string) =>
      ipcRenderer.invoke("db:save-assembly", projectId, text),
  },
});
```

**Migration from localStorage:**

```typescript
// Run once on app startup
async function migrateFromLocalStorage() {
  const oldProjects = localStorage.getItem("microtrace_projects");
  if (!oldProjects) return;

  const projects = JSON.parse(oldProjects);
  for (const project of projects) {
    await window.api.db.saveProject(project);
  }

  const activeId = localStorage.getItem("microtrace_active_project");
  if (activeId) {
    await window.api.db.setActiveProject(activeId);
  }

  // Clear old data
  localStorage.removeItem("microtrace_projects");
  localStorage.removeItem("microtrace_active_project");
}
```

#### 11. **OS-Native Configuration Files**

**Advantages over electron-store:**

- ✅ No dependency issues
- ✅ Standard Electron API
- ✅ Platform-specific locations
- ✅ Simple JSON file
- ✅ Easy backup/restore

**Implementation:**

```typescript
// src/services/config.ts
import { app } from "electron";
import { readFileSync, writeFileSync, existsSync, mkdirSync } from "fs";
import { join } from "path";

interface AppConfig {
  ghidraPath: string;
  theme: "dark" | "light";
  pythonPath?: string;
  yaraRulesPath?: string;
  autoSave: boolean;
  maxRecentProjects: number;
}

const defaultConfig: AppConfig = {
  ghidraPath: "",
  theme: "dark",
  autoSave: true,
  maxRecentProjects: 10,
};

class ConfigManager {
  private configPath: string;
  private config: AppConfig;

  constructor() {
    // Platform-specific paths:
    // Windows: C:\Users\{user}\AppData\Roaming\MicroTrace\config.json
    // macOS: ~/Library/Application Support/MicroTrace/config.json
    // Linux: ~/.config/MicroTrace/config.json
    const userDataPath = app.getPath("userData");
    this.configPath = join(userDataPath, "config.json");
    this.config = this.load();
  }

  private load(): AppConfig {
    if (!existsSync(this.configPath)) {
      this.save(defaultConfig);
      return defaultConfig;
    }

    try {
      const data = readFileSync(this.configPath, "utf-8");
      return { ...defaultConfig, ...JSON.parse(data) };
    } catch (error) {
      console.error("Failed to load config:", error);
      return defaultConfig;
    }
  }

  private save(config: AppConfig): void {
    const dir = join(this.configPath, "..");
    if (!existsSync(dir)) {
      mkdirSync(dir, { recursive: true });
    }

    writeFileSync(this.configPath, JSON.stringify(config, null, 2), "utf-8");
  }

  get<K extends keyof AppConfig>(key: K): AppConfig[K] {
    return this.config[key];
  }

  set<K extends keyof AppConfig>(key: K, value: AppConfig[K]): void {
    this.config[key] = value;
    this.save(this.config);
  }

  getAll(): AppConfig {
    return { ...this.config };
  }

  reset(): void {
    this.config = defaultConfig;
    this.save(this.config);
  }
}

export const configManager = new ConfigManager();
```

**IPC Integration:**

```typescript
// main.ts
import { configManager } from "./services/config";

ipcMain.handle("config:get", async (_, key: string) => {
  return configManager.get(key as keyof AppConfig);
});

ipcMain.handle("config:set", async (_, key: string, value: any) => {
  configManager.set(key as keyof AppConfig, value);
  return { success: true };
});

ipcMain.handle("config:get-all", async () => {
  return configManager.getAll();
});

ipcMain.handle("config:reset", async () => {
  configManager.reset();
  return { success: true };
});
```

#### 12. **Performance Optimization**

```typescript
// Virtualize large lists
import { FixedSizeList } from "react-window";

// Memoize expensive computations
const memoizedGraph = useMemo(() => computeGraph(data), [data]);

// Lazy load components
const Dashboard = lazy(() => import("./Dashboard"));

// Web Workers for heavy processing
const worker = new Worker("analysis-worker.js");
worker.postMessage({ data });
worker.onmessage = (e) => updateUI(e.data);
```

---

## Dependencies

### Node.js Dependencies (`package.json`)

```json
{
  "name": "microtrace",
  "version": "1.0.0",
  "type": "module",
  "description": "Embedded Trace Analysis Tool",
  "main": "dist/main.js",
  "scripts": {
    "build": "tsc -p tsconfig.main.json && tsc -p tsconfig.renderer.json",
    "start": "npm run build && electron ."
  },
  "devDependencies": {
    "@types/chart.js": "^2.9.41",
    "@types/d3": "^7.4.3",
    "@types/node": "^24.10.1",
    "electron": "^39.2.3",
    "ts-node": "^10.9.2",
    "typescript": "^5.9.3"
  },
  "dependencies": {
    "chart.js": "^4.4.0",
    "d3": "^7.9.0",
    "better-sqlite3": "^9.2.2"
  }
}
```

### Python Dependencies (`requirements.txt`)

```
# Currently minimal
yara-python
ghidra-bridge  # Planned
requests       # For API calls
```

**Recommended Additions:**

```
pefile         # PE file parsing
capstone       # Disassembly
r2pipe         # Radare2 integration
cve-search     # CVE database
python-magic   # File type detection
pytest         # Testing
black          # Code formatting
mypy           # Type checking
```

---

## Setup & Configuration

### Development Setup

**1. Install Prerequisites:**

```bash
# Node.js 18+ and npm
node --version
npm --version

# Python 3.8+
python --version
pip --version

# Git
git --version
```

**2. Clone and Install:**

```bash
git clone <repository-url>
cd Grad-project
npm install
cd python && pip install -r requirements.txt
```

**3. Build:**

```bash
npm run build
```

**4. Run:**

```bash
npm start
```

### Configuration Files

**`tsconfig.main.json`** (Main Process)

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ES2020",
    "moduleResolution": "node",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/main.ts", "src/preload.ts", "src/types/**/*"],
  "exclude": ["node_modules"]
}
```

**`tsconfig.renderer.json`** (Renderer Process)

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ES2020",
    "moduleResolution": "node",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "lib": ["ES2020", "DOM"]
  },
  "include": [
    "src/renderer.ts",
    "src/dashboard.ts",
    "src/upload.ts",
    "src/functionGraph.ts",
    "src/analyzerTabs.ts",
    "src/settings.ts",
    "src/theme.ts",
    "src/types/**/*"
  ],
  "exclude": ["node_modules", "src/main.ts", "src/preload.ts"]
}
```

**`.gitignore`**

```
node_modules/
dist/
python/venv/
python/__pycache__/
*.pyc
.DS_Store
.env
```

---

## Detailed Module Documentation

### `main.ts` - Main Process

**Entry Point:** Electron main process

**Responsibilities:**

1. Create browser window
2. Manage Python subprocess
3. Handle IPC communication
4. Store settings (in-memory)

**Key Functions:**

#### `createWindow()`

```typescript
function createWindow(): void;
```

Creates the main browser window with security settings.

**Configuration:**

- Width: 1200px
- Height: 800px
- Preload script: `dist/preload.js`
- Context isolation: enabled
- Node integration: disabled

**Security Settings:**

```javascript
webPreferences: {
  preload: path.join(__dirname, "preload.js"),
  contextIsolation: true,
  nodeIntegration: false,
}
```

#### Python Subprocess Management

```typescript
py = spawn("python", ["-u", path.join(__dirname, "../python/engine.py")], {
  stdio: ["pipe", "pipe", "pipe"],
});
```

**Event Handlers:**

- `py.stdout.on("data")` - Forward to renderer
- `py.stderr.on("data")` - Log errors
- `py.on("error")` - Handle spawn errors
- `py.on("exit")` - Handle unexpected termination

**IPC Handlers:**

#### `py-in` - Send Command to Python

```typescript
ipcMain.handle("py-in", async (_event, text: string) => {
  if (!py || !py.stdin || py.killed) {
    return { success: false, error: "Python not available" };
  }

  try {
    const command = text.trim() + "\n";
    py.stdin.write(command);
    return { success: true };
  } catch (error) {
    return { success: false, error: error.message };
  }
});
```

#### Settings Handlers

```typescript
ipcMain.handle("settings-get", async (_event, key: keyof Settings) => {
  return settingsStore[key];
});

ipcMain.handle("settings-set", async (_event, key, value) => {
  settingsStore[key] = value;
  return { success: true };
});
```

---

### `functionGraph.ts` - Visualization

**Purpose:** Interactive D3.js force-directed graph

**Key Components:**

#### Data Loading

```typescript
async function loadData(): Promise<GraphData> {
  const res = await fetch("../assets/functionData2.json");
  return res.json();
}
```

#### Graph Mounting

```typescript
function mountGraph(data: GraphData): void {
  // 1. Setup SVG canvas
  const svg = d3.select("#graph");
  const width = wrap.clientWidth;
  const height = wrap.clientHeight;

  // 2. Create force simulation
  const sim = d3
    .forceSimulation(nodes)
    .force("link", d3.forceLink(links).distance(110))
    .force("charge", d3.forceManyBody().strength(-350))
    .force("center", d3.forceCenter(width / 2, height / 2))
    .force("collide", d3.forceCollide().radius(NODE_RADIUS * 2.2));

  // 3. Add zoom/pan behavior
  const zoom = d3.zoom().scaleExtent([0.2, 4]);
  svg.call(zoom);

  // 4. Render links (with arrows)
  const link = container
    .append("g")
    .selectAll("path")
    .data(links)
    .enter()
    .append("path")
    .attr("marker-end", "url(#arrow)");

  // 5. Render nodes (circles + labels)
  const node = container
    .append("g")
    .selectAll("g")
    .data(nodes)
    .enter()
    .append("g")
    .call(drag);

  node
    .append("circle")
    .attr("r", NODE_RADIUS)
    .attr("fill", (d) => COLORS[d.type]);

  node.append("text").text((d) => d.name);

  // 6. Add interaction handlers
  node
    .on("mouseover", highlightNeighbors)
    .on("mouseout", resetHighlight)
    .on("click", showDetails);

  // 7. Simulation tick
  sim.on("tick", updatePositions);
}
```

#### Drag Behavior

```typescript
const drag = d3
  .drag()
  .on("start", (event, d) => {
    if (!event.active) sim.alphaTarget(0.3).restart();
    d.fx = d.x;
    d.fy = d.y;
  })
  .on("drag", (event, d) => {
    // Constrain to bounds
    d.fx = Math.max(minX, Math.min(maxX, event.x));
    d.fy = Math.max(minY, Math.min(maxY, event.y));
  })
  .on("end", (event, d) => {
    if (!event.active) sim.alphaTarget(0);
    // Keep node pinned: d.fx/fy remain set
  });
```

#### Filtering

```typescript
function applyFilters(): void {
  // Type filter
  node.style("display", (d: NodeData) => {
    if (!active.has(d.type)) return "none";

    // Depth filter
    if (depthFilterEnabled && maxDepth !== null) {
      if (d.depth > maxDepth) return "none";
    }

    return null;
  });

  // Hide links where either endpoint is hidden
  link.style("display", (lk: any) => {
    const sn = nodes.find((n) => n.id === lk.source.id);
    const tn = nodes.find((n) => n.id === lk.target.id);
    if (!sn || !tn) return "none";
    if (!active.has(sn.type) || !active.has(tn.type)) return "none";
    // ... depth check
    return null;
  });
}
```

---

### `analyzerTabs.ts` - Code Viewer

**Key Features:**

- Lazy loading of assembly file
- Syntax highlighting (regex-based)
- Tab switching

#### Assembly File Loading with Caching

```typescript
let assemblyFileCache: string | null = null;

async function loadAssemblyFile(): Promise<string> {
  if (assemblyFileCache) {
    return assemblyFileCache;
  }

  const response = await fetch("../assets/asm.txt");
  assemblyFileCache = await response.text();
  return assemblyFileCache;
}
```

#### Extract Function Assembly

```typescript
async function getAssemblyForFunction(
  startLine: number,
  instructionCount: number
): Promise<string> {
  const fullAsm = await loadAssemblyFile();
  const lines = fullAsm.split("\n");
  const extracted = lines.slice(startLine, startLine + instructionCount);
  return extracted.join("\n") || "No assembly code found";
}
```

#### Syntax Highlighting

```typescript
function applySyntaxHighlight(text: string, language: "asm" | "c"): string {
  let highlighted = text;

  if (language === "asm") {
    // Comments: ;.*
    highlighted = highlighted.replace(
      /;.*/g,
      '<span class="comment">$&</span>'
    );

    // Keywords: mov, ldr, etc.
    highlighted = highlighted.replace(
      /\b(mov|ldr|str|add|sub|push|pop|bl?|bx)\b/gi,
      '<span class="keyword">$&</span>'
    );

    // Hex numbers: 0x[0-9a-f]+
    highlighted = highlighted.replace(
      /0x[0-9a-f]+/gi,
      '<span class="number">$&</span>'
    );

    // Registers: r0-r15
    highlighted = highlighted.replace(
      /\br\d+\b/gi,
      '<span class="function">$&</span>'
    );
  } else if (language === "c") {
    // Line comments: //.*
    highlighted = highlighted.replace(
      /\/\/.*/g,
      '<span class="comment">$&</span>'
    );

    // Block comments: /* ... */
    highlighted = highlighted.replace(
      /\/\*[\s\S]*?\*\//g,
      '<span class="comment">$&</span>'
    );

    // Keywords
    highlighted = highlighted.replace(
      /\b(if|else|return|int|void|undefined)\b/g,
      '<span class="keyword">$&</span>'
    );

    // Strings
    highlighted = highlighted.replace(
      /"[^"]*"/g,
      '<span class="string">$&</span>'
    );
  }

  return highlighted;
}
```

---

### `dashboard.ts` - Analytics

**Chart Creation:**

```typescript
function createCharts(data: AnalysisData): void {
  const isLightTheme = document.body.dataset.lightTheme === "true";
  const textColor = isLightTheme ? "#1a1a2e" : "#e5e7eb";
  const gridColor = isLightTheme
    ? "rgba(0, 0, 0, 0.1)"
    : "rgba(255, 255, 255, 0.1)";

  const chartOptions = {
    responsive: true,
    maintainAspectRatio: true,
    plugins: {
      legend: {
        labels: { color: textColor },
      },
    },
    scales: {
      x: {
        ticks: { color: textColor },
        grid: { color: gridColor },
      },
      y: {
        ticks: { color: textColor },
        grid: { color: gridColor },
      },
    },
  };

  // Layer Distribution (Doughnut)
  const layerCtx = document.getElementById("layerChart").getContext("2d");
  charts.layerChart = new Chart(layerCtx, {
    type: "doughnut",
    data: {
      labels: Object.keys(data.layerDistribution),
      datasets: [
        {
          data: Object.values(data.layerDistribution),
          backgroundColor: [
            "rgba(31, 119, 180, 0.8)", // MCAL
            "rgba(255, 127, 14, 0.8)", // HAL
            "rgba(107, 174, 214, 0.8)", // Other
            "rgba(152, 223, 138, 0.8)", // Library
          ],
          borderWidth: 2,
          borderColor: textColor,
        },
      ],
    },
    options: chartOptions,
  });

  // ... other charts (Bar, Pie, Line)
}
```

**Theme Observer:**

```typescript
const themeObserver = new MutationObserver(() => {
  // Destroy and recreate charts on theme change
  Object.keys(charts).forEach((key) => {
    charts[key].destroy();
  });
  charts = {};

  if (resultsSection.style.display !== "none") {
    createCharts(sampleData);
  }
});

themeObserver.observe(document.body, {
  attributes: true,
  attributeFilter: ["data-light-theme"],
});
```

---

### `upload.ts` - Project Management

**Project Rendering:**

```typescript
function renderProjects(): void {
  const list = document.getElementById("projectList");

  if (projects.length === 0) {
    list.innerHTML = "<p>No projects added yet.</p>";
    return;
  }

  list.innerHTML = "";

  projects.forEach((p) => {
    const item = document.createElement("div");
    item.className = `project-item ${p.id === activeProjectId ? "active" : ""}`;

    const date = new Date(p.dateAdded).toLocaleDateString();

    item.innerHTML = `
      <div class="project-info">
        <h4>${p.name} ${p.id === activeProjectId ? "(Active)" : ""}</h4>
        <p>Path: ${p.path}</p>
        <p>CPU: ${p.cpuFamily} | Flash: ${p.flashAddress} | Added: ${date}</p>
      </div>
      <div class="project-actions">
        ${
          p.id !== activeProjectId
            ? `<button class="btn-activate" data-id="${p.id}">Activate</button>`
            : ""
        }
        <button class="btn-delete" data-id="${p.id}">Delete</button>
      </div>
    `;

    list.appendChild(item);
  });

  // Attach event listeners
  list.querySelectorAll(".btn-activate").forEach((btn) => {
    btn.addEventListener("click", (e) => {
      const id = e.target.dataset.id;
      if (id) setActiveProject(id);
    });
  });

  list.querySelectorAll(".btn-delete").forEach((btn) => {
    btn.addEventListener("click", (e) => {
      const id = e.target.dataset.id;
      if (id && confirm("Delete this project?")) {
        deleteProject(id);
      }
    });
  });
}
```

---

## Future Enhancements

### Planned Features

1. **Ghidra Integration**

   - Automated headless analysis
   - Decompilation pipeline
   - Symbol extraction

2. **YARA Rule Engine**

   - MCAL/HAL pattern detection
   - Library fingerprinting
   - Custom rule creation UI

3. **CVE Database Integration**

   - NVD API integration
   - Automated vulnerability scanning
   - Severity reporting

4. **SBOM Export**

   - SPDX format support
   - CycloneDX format support
   - Custom report templates

5. **AI Assistant (RAG)**

   - ARM datasheet querying
   - Natural language function analysis
   - LangChain/LlamaIndex integration

6. **Advanced Visualization**

   - 3D graph view
   - Call stack visualization
   - Memory map viewer

7. **Collaboration Features**

   - Project sharing
   - Cloud storage integration
   - Team annotations

8. **Plugin System**
   - Custom analyzers
   - Export formats
   - UI extensions

---

## Appendix

### Glossary

- **SBOM**: Software Bill of Materials
- **MCAL**: Microcontroller Abstraction Layer
- **HAL**: Hardware Abstraction Layer
- **CVE**: Common Vulnerabilities and Exposures
- **YARA**: Pattern matching tool for malware research
- **IPC**: Inter-Process Communication
- **RAG**: Retrieval-Augmented Generation

### External Resources

- [Electron Documentation](https://www.electronjs.org/docs)
- [D3.js Documentation](https://d3js.org/)
- [Chart.js Documentation](https://www.chartjs.org/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Ghidra Documentation](https://ghidra-sre.org/)
- [YARA Documentation](https://yara.readthedocs.io/)

### License Considerations

This project uses:

- Electron (MIT)
- D3.js (ISC)
- Chart.js (MIT)
- TypeScript (Apache 2.0)
- Ghidra (Apache 2.0) - External tool

**Ensure compliance** with all license requirements when redistributing.

---

## Contact & Support

For questions or contributions, contact the development team:

- Email: contact@microtrace.dev
- GitHub: [omarsawaf1](https://github.com/omarsawaf1)
- LinkedIn: [Omar Mohamed Hatem](https://www.linkedin.com/in/omar-mohamed-hatem-057a8025b/)

---

**END OF DOCUMENTATION**

_Version: 1.0_  
_Last Updated: 2025-11-25_  
_Generated for Project Recreation_
