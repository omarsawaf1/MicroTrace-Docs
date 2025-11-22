# SVD Database Documentation

## Overview

**Purpose:**
This module parses ARM CMSIS SVD files and reference-manual PDFs for an STM32 microcontroller family. It extracts peripherals, registers, and fields, and stores a flattened, query-friendly representation in **MongoDB**.

The dataset allows quick lookup of register addresses, field bit offsets/widths, and textual descriptions for use in tooling, documentation generation, or verification.

**Target Database:**
MongoDB Atlas. The code uses `pymongo` and expects a standard connection URI. The main collection stores one document per register-field pair, or one document per register if no fields are present.

---

## Imports Overview

These Python modules provide the required functionality:

- **os, re, io, copy** – utility and file operations
- **xml.etree.ElementTree** – parsing SVD XML files
- **pymongo** – MongoDB database connectivity
- **fitz (PyMuPDF)** – extracting text and images from PDFs
- **pytesseract + PIL** – OCR for unreadable PDF pages

**Code:**

```python
import os, sys, re, io, copy
import xml.etree.ElementTree as ET
from pymongo import MongoClient
import fitz, pytesseract
from PIL import Image
```

---

## MongoDB Connection

This function connects to MongoDB Atlas, verifies connectivity, and returns the collection object for inserting documents.

```python
def connect():
    c = MongoClient(MONGO_URI, tls=True)
    c.admin.command("ping")
    return c[DB][COL]
```

---

## PDF Base Address Extraction

Reads the reference manual PDF and extracts base addresses for peripherals. OCR is used when a PDF page contains little or no readable text.

```python
def extract_pdf_bases(path):
    if not os.path.exists(path):
        return {}
    txt = ""
    for pg in fitz.open(path):
        t = pg.get_text()
        if len(t.strip()) < 40:
            img = Image.open(io.BytesIO(pg.get_pixmap(dpi=200).tobytes("png")))
            t = pytesseract.image_to_string(img)
        txt += t + "\n"
    bases = {}
    for m in re.finditer(r"([A-Z][A-Z0-9_]+)[^\w]{0,10}(0x[0-9A-Fa-f]{6,8})", txt):
        bases[m.group(1)] = m.group(2).upper()
    return bases
```

---

## Peripheral Expansion (Handling `derivedFrom`)

SVD allows peripherals to inherit structure from others using `derivedFrom`. This function recursively expands inherited peripherals and merges fields correctly.

```python
def expand(peripherals, name, seen=None):
    if name not in peripherals:
        return None
    seen = seen or set()
    if name in seen:
        return None
    seen.add(name)
    node = peripherals[name]["raw"]
    base = peripherals[name]["derived"]
    if base:
        bnode = expand(peripherals, base, seen)
        if bnode is not None:
            m = copy.deepcopy(bnode)
            for ch in node:
                if ch.tag in ["registers","clusters"]:
                    for old in m.findall(ch.tag):
                        m.remove(old)
                m.append(copy.deepcopy(ch))
            m.find("name").text = node.findtext("name")
            m.find("baseAddress").text = node.findtext("baseAddress", m.findtext("baseAddress","0x0"))
            return m
    return node
```

---

## SVD Parsing

The main logic reads SVD peripherals, registers, clusters, and fields. It calculates addresses, merges PDF bases, and produces MongoDB-ready documents.

```python
def parse_svd(path, pdf_bases):
    tree = ET.parse(path)
    root = tree.getroot().find("peripherals")
    per = {p.findtext("name"):{"raw":p,"derived":p.get("derivedFrom")} for p in root}
    docs = []

    def add(reg, P, desc, bint, bhex, pref=""):
        rname = pref + reg.findtext("name","").upper()
        off = int(reg.findtext("addressOffset","0x0"),0)
        hexaddr = hex(bint+off).upper()
        fields = reg.find("fields")
        if fields is None:
            docs.append({
                "PERIPHERAL": P,
                "DESCRIPTION": desc,
                "BASEADDRESS": bhex,
                "REGISTER": rname,
                "REGISTER_DESCRIPTION": reg.findtext("description",""),
                "ADDRESSOFFSET": hex(off).upper(),
                "RESETVALUE": reg.findtext("resetValue","0"),
                "HEXADDRESS": hexaddr,
                "FIELD": None,
                "FIELD_DESCRIPTION": None,
                "BITOFFSET": None,
                "BITWIDTH": None
            })
        else:
            for f in fields.findall("field"):
                docs.append({
                    "PERIPHERAL": P,
                    "DESCRIPTION": desc,
                    "BASEADDRESS": bhex,
                    "REGISTER": rname,
                    "REGISTER_DESCRIPTION": reg.findtext("description",""),
                    "ADDRESSOFFSET": hex(off).upper(),
                    "RESETVALUE": reg.findtext("resetValue","0"),
                    "HEXADDRESS": hexaddr,
                    "FIELD": f.findtext("name",""),
                    "FIELD_DESCRIPTION": f.findtext("description",""),
                    "BITOFFSET": int(f.findtext("bitOffset","0")),
                    "BITWIDTH": int(f.findtext("bitWidth","0"))
                })

    for name in per:
        node = expand(per, name)
        if node is None:
            continue
        P = node.findtext("name","N/A").upper()
        base = pdf_bases.get(P, node.findtext("baseAddress","0x0"))
        bint = int(base,0)
        bhex = hex(bint).upper()
        desc = node.findtext("description","")

        regs = node.find("registers")
        if regs:
            for r in regs.findall("register"):
                add(r, P, desc, bint, bhex)

        for cl in node.findall(".//cluster"):
            cpre = cl.findtext("name","").upper()+"_"
            coff = int(cl.findtext("addressOffset","0x0"),0)
            for r in cl.findall("register"):
                add(r, P, desc, bint+coff, bhex, cpre)

    return docs
```

---

## MongoDB Insertion Workflow

The `main()` function runs the complete pipeline: connects to the database, extracts PDF bases, parses SVD, clears old documents, and inserts the new structured dataset.

```python
def main():
    col = connect()
    bases = extract_pdf_bases(PDF)
    docs = parse_svd(SVD, bases)
    col.delete_many({})
    col.insert_many(docs)
    print("DONE:", len(docs), "records uploaded.")

if __name__ == "__main__":
    main()
```

---

## Database Example

**Collection:** `stm32_registers_full`

```json
{
  "PERIPHERAL": "FLASH",
  "DESCRIPTION": "Flash memory control registers",
  "BASEADDRESS": "0X40023C00",
  "REGISTER": "ACR",
  "REGISTER_DESCRIPTION": "Flash access control register",
  "ADDRESSOFFSET": "0X0",
  "RESETVALUE": "0X20",
  "HEXADDRESS": "0X40023C00",
  "FIELD": "LATENCY",
  "FIELD_DESCRIPTION": "Latency",
  "BITOFFSET": 0,
  "BITWIDTH": 3
}
```

**Screenshot:**
_Include a screenshot of the MongoDB document view for clarity._

<img src="../assets/mongoDB.png" alt="MongoDB Logo" 
     style="display:block; margin:auto; width:80%; max-width:800px; border:2px solid #4c7caff; border-radius:8px; padding:5px;">
