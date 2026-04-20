# Backend

## Purpose

The backend coordinates:

- project creation and deletion
- firmware analysis execution
- Ghidra settings persistence
- graph and analysis retrieval
- register lookup

## Main Modules

| Module | Responsibility |
| --- | --- |
| `main.py` | app factory and service wiring |
| `api/routes/` | FastAPI endpoints |
| `services/analysis_service.py` | orchestrates analysis |
| `services/ghidra_service.py` | launches Ghidra headless |
| `services/result_store.py` | stores artifacts |
| `services/register_lookup_service.py` | Elasticsearch-backed register search |
| `repositories/` | settings and project persistence |

## Application Wiring

The backend uses a service container pattern assembled in `create_app()`.

```python
app = FastAPI(
    title="MicroTrace Firmware Backend",
    version="1.0.0",
    lifespan=lifespan,
)
app.state.services = services
```

## Project Analysis Flow

```python
run = services.analysis_service.run_project_analysis(project_id)
```

That call:

1. loads the project
2. resolves the effective Ghidra path
3. creates run artifacts
4. calls Ghidra export
5. parses outputs into graph-ready payloads
6. stores and indexes run state

## Artifact Layout

Each analysis run creates:

```text
<output>/<project_id>/<analysis_id>/
  analysis.json
  decompiled_code.txt
  listing.txt
```

## Example Endpoints

```text
GET  /api/v1/health
GET  /api/v1/settings
PUT  /api/v1/settings/ghidra
GET  /api/v1/projects
POST /api/v1/projects
POST /api/v1/projects/{project_id}/analyze
GET  /api/v1/projects/{project_id}/graph
GET  /api/v1/registers
```
