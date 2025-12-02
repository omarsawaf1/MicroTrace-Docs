# Headless Ghidra ARM Firmware Analyzer

A headless Ghidra automation project that loads firmware, analyzes it, and exports both full disassembly and decompiled C code into text files.

---

## Overview

This project automates the analysis of ARM embedded firmware using Ghidra's headless mode. It eliminates the need for manual GUI interaction by programmatically importing firmware, setting the correct processor architecture, analyzing the binary, and exporting comprehensive function listings in both assembly and decompiled pseudo-C formats.

---

## Features

- **Fully Automated:** No GUI required—runs entirely in headless mode  
- **ARM Firmware Support:** Optimized for ARM Cortex-M microcontrollers (configurable processor strings)  
- **Dual Export Format:**  
     - Assembly listings for every discovered function  
     - Decompiled pseudo-C code for every function  
- **Function Index:** JSON index file with function names, entry addresses, and sizes  
- **Smart Base Detection:** Automatically detects image base from vector table (optional manual override)  
- **Cross-Platform:** Works on Windows, Linux, and macOS  

---

## Prerequisites

- **Ghidra:** Installed and accessible (version 10.0+ recommended)  
     - Must have path to `analyzeHeadless` (Linux/Mac) or `analyzeHeadless.bat` (Windows)  
- **Java:** Version compatible with your Ghidra installation (typically Java 11+)  
- **Python 3:** Python 3.6 or higher (uses only standard library)  

---

## Project Structure
```
 Headless-Ghidra-ARM-Firmware-Analyzer (short)

 Small toolkit to analyze ARM firmware using Ghidra headless output and MCU SVD register metadata.

 This repository provides three main components:

 IntialDB.py — parse an SVD and build an SQLite DB (`Data\\mydb_full.db`) of registers/fields.  
 export_disasm.py — Ghidra headless Jython script to export a full disassembly (`code.txt`) and per-function
                  C/ASM output (`asm.txt`).  
 analyze_firmware.py — Python analyzer that parses `code.txt`/`asm.txt`, matches pointer values to registers
                    in the DB, classifies functions as `MCAL` or `Other`, and writes`firmware_analysis_enhanced.json` (graph + annotations).  
```
## Quick workflow

### 1. Build the register DB (from SVD)
```
 from IntialDB import create_db, parse_svd, upload
 conn, cur = create_db()
 rows = parse_svd(r"Data\\stm32f407.SVD")
 upload(rows, cur, conn)
 conn.close()
```
### 2. Run Ghidra headless to generate `code.txt` and `asm.txt`:

```
 "C:\\Program Files\\ghidra\\support\\analyzeHeadless" C:\\tmp\\proj projname -import C:\\path\\firmware.bin -scriptPath C:\\path\\to\\scripts -postScript export_disasm.py 0x08000000 0x0 "C:\\out\\code.txt" "C:\\out\\asm.txt" "C:\\path\\firmware.bin" -deleteProject
```
### 3. Run the analyzer (reads `code.txt` and `asm.txt` produced by `export_disasm.py`):

```
 python analyze_firmware.py
```
## Output
* `firmware_analysis_enhanced.json `— analysis summary, nodes (functions) with      `data_calls` annotated by `Peripheral`/`Register`, function `type` (`MCAL` or `Other`), and `links` (call graph).

## Requirements
* Python 3.8+
* Ghidra (for export_disasm.py) and a matching Java runtime
* SVD file for your MCU in Data/ (default Data/stm32f407.SVD)

## See DOCUMENTATION.md for full details, examples, and troubleshooting tips.
 Keep Project for Manual Inspection
```
               bash
 python run_ghidra_headless.py firmware.bin \
    --ghidra /opt/ghidra/support/analyzeHeadless \
    --outdir ./output \
    --keep-project
```
