# Headless Ghidra ARM Firmware Analyzer

A headless Ghidra automation project that loads firmware, analyzes it, and exports both full disassembly and decompiled C code into text files.

---

## Overview

The Headless Ghidra ARM Firmware Analyzer automates ARM Cortex-M firmware analysis using Ghidra’s headless mode.

This eliminates GUI interaction and produces:

 - A full disassembly dump `code.txt`

 - Decompiled C-like pseudo code `asm.txt`

 - A structured firmware analysis JSON `firmware_analysis_enhanced.json`

 - Optional SQLite database using SVD metadata to identify hardware registers

It is cross-platform and works on:

 - Windows

 - Linux

 - macOS

---

## Features

- **Fully Automated:** ` No GUI required ` Everything runs via Ghidra headless scripts and Python.  
- **ARM Firmware Support:** Optimized for ARM Cortex-M microcontrollers `configurable processor strings  
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
- **Optional:** An SVD file for your microcontroller `Data/stm32f407.SVD` by default
---

=== "Project Structure"

``` markdown
Small toolkit to analyze ARM firmware using Ghidra headless output and MCU SVD register metadata.
This repository provides three main components:
 * IntialDB.py          → parse an SVD and build an SQLite DB (`Data\\mydb_full.db`) of registers/fields.  
 * export_disasm.py     → Ghidra headless Jython script to export a full disassembly (`code.txt`) and 
                          per-function C/ASM output (`asm.txt`).
 * analyze_firmware.py  → Python analyzer that parses `code.txt`/`asm.txt`, matches pointer values to registers
                          in the DB, classifies functions as `MCAL` or `Other`, and writes
                          `firmware_analysis_enhanced.json` (graph + annotations). 
 * Data/                  → Contains SVD and database files
 * SAVE/                  → Output folders

```
## Quick workflow

=== "Step 1. Build the register DB (from SVD)"

``` python

from IntialDB import create_db, parse_svd, upload
 conn, cur = create_db()
 rows = parse_svd(r"Data\\stm32f407.SVD")
 upload(rows, cur, conn)
 conn.close()
```
=== "Step 2. Run Ghidra headless to generate `code.txt` and `asm.txt`:"

``` python

"C:\\Program Files\\ghidra\\support\\analyzeHeadless" C:\\tmp\\proj projname -import C:\\path\\firmware.bin -scriptPath C:\\path\\to\\scripts -postScript export_disasm.py 0x08000000 0x0 "C:\\out\\code.txt" "C:\\out\\asm.txt" "C:\\path\\firmware.bin" -deleteProject
```
=== "Step 3. Run the analyzer (reads `code.txt` and `asm.txt` produced by `export_disasm.py`):"

``` python

python analyze_firmware.py
```
## Output
* `firmware_analysis_enhanced.json `— analysis summary, nodes (functions) with      `data_calls` annotated by `Peripheral`/`Register`, function `type` (`MCAL` or `Other`), and `links` (call graph).

=== "See Headless-Ghidra-ARM-Firmware-Analyzer.md for full details, examples, and troubleshooting tips."
 Keep Project for Manual Inspection

``` bash
 python run_ghidra_headless.py firmware.bin/
    --ghidra /opt/ghidra/support/analyzeHeadless/
    --outdir ./output/
    --keep-project
```
=== "Troubleshooting"
Common issues:

- No data_calls detected

    * Ensure `asm.txt` contains pointer metadata (VALUE, REFS).

- Missing memory blocks

    * Specify -loader BinaryLoader -loader-baseAddr.

- DB not matching addresses

    * Check that pointer values are normalized `0x40023800` format.


