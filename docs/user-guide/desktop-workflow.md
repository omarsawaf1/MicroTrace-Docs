# Desktop Workflow

## Main User Journey

1. Open the app
2. Configure Ghidra in settings if needed
3. Upload a firmware project
4. Run analysis
5. Inspect the graph, assembly, and saved results
6. Open the MCP workspace for AI-assisted exploration

## Settings Flow

The settings screen lets the user set the Ghidra path for local and desktop runs.

Expected executable names:

- `analyzeHeadless`
- `analyzeHeadless.bat`

## Create a Project

The upload flow expects:

- project name
- firmware path
- flash address such as `0x08000000`
- CPU family such as `arm:le:32:v7m`

## Analyze a Project

When the user clicks analyze, the backend:

- validates the configured Ghidra path
- creates an output directory
- runs Ghidra headless export
- parses and stores the resulting artifacts

## Outputs Generated

For each analysis run, MicroTrace stores:

- `listing.txt`
- `decompiled_code.txt`
- `analysis.json`

## Viewing Results

The function graph view supports:

- graph navigation
- assembly display
- decompiled code display
- AI drawer integration

## Example Health Response

```json
{
  "ok": true,
  "app": "microtrace-backend",
  "settings": {
    "configured": true,
    "source": "file",
    "ghidra_path": "D:\\Tools\\ghidra\\support\\analyzeHeadless.bat"
  }
}
```
