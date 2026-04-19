# API Reference

## Backend API

### Health

```http
GET /api/v1/health
```

### Settings

```http
GET /api/v1/settings
PUT /api/v1/settings/ghidra
```

Example request:

```json
{
  "ghidra_path": "D:\\Tools\\ghidra\\support\\analyzeHeadless.bat"
}
```

### Projects

```http
GET    /api/v1/projects
POST   /api/v1/projects
DELETE /api/v1/projects/{project_id}
POST   /api/v1/projects/{project_id}/analyze
GET    /api/v1/projects/{project_id}/analysis
GET    /api/v1/projects/{project_id}/graph
GET    /api/v1/projects/{project_id}/asm
GET    /api/v1/projects/{project_id}/analysis/raw
```

Example project creation:

```json
{
  "name": "demo",
  "firmware_path": "C:\\firmware\\demo.bin",
  "flash_address": "0x08000000",
  "cpu_family": "arm:le:32:v7m"
}
```

### Registers

```http
GET /api/v1/registers?q=USART&size=20
```

Query parameters:

| Parameter | Purpose |
| --- | --- |
| `base` | peripheral base filter |
| `hex_address` | address filter |
| `register` | register name filter |
| `field` | field name filter |
| `q` | free-text query |
| `size` | result count |

## MCP API

### Health and status

```http
GET /health
GET /status
```

### Chat and tools

```http
POST /connect
POST /query
GET  /tools
POST /conversation/clear
GET  /conversation/history
```

### LLM settings

```http
GET  /llm/config
POST /llm/config
GET  /llm/models
```
