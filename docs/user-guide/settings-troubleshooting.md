# Settings and Troubleshooting

## Ghidra Path Validation Rules

The backend validates that the configured Ghidra executable:

- is an absolute path
- points to a file
- has a valid name such as `analyzeHeadless` or `analyzeHeadless.bat`

## Common Problems

### Analysis returns `409`

Cause:
- Ghidra path is not configured

Fix:
- open settings and save the correct Ghidra executable path

### Analysis returns `422`

Cause:
- path exists in settings but is invalid

Fix:
- reselect the executable
- ensure it points to the actual Ghidra headless launcher

### Graph is empty

Cause:
- analysis artifacts were not generated
- parsing produced no functions

Fix:
- inspect backend logs
- inspect `listing.txt` and `decompiled_code.txt`

### MCP UI says API offline

Cause:
- MCP FastAPI service is not running

Fix:
- verify port `8001`
- verify Electron or Docker started the MCP service

## Troubleshooting Checklist

- Can the backend health endpoint load?
- Is Ghidra configured?
- Was `analysis.json` written?
- Is Elasticsearch reachable?
- Is the MCP service healthy?
