# MicroTrace Docs

This repository contains the MkDocs Material documentation site for MicroTrace.

## Local Preview

Create a virtual environment, install MkDocs Material, and run the local server.

```bash
python -m venv .venv
```

### Windows

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m mkdocs serve
```

### Linux or macOS

```bash
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m mkdocs serve
```

## Main Docs Structure

- `docs/index.md`: site landing page
- `docs/getting-started/`, `docs/user-guide/`, `docs/developer-guide/`, `docs/reference/`: main documentation sections
- `docs/github-submodules.md`: git submodule guide
- `mkdocs.yml`: Material navigation and theme configuration
- `requirements.txt`: Python dependencies for the documentation site
