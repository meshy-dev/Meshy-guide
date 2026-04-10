# Meshy 3D Printing Workflow

Generate 3D-printable models from text or images using the Meshy API, then export directly to your slicer.

This repository provides a complete, runnable pipeline from prompt to physical object.

---

## Overview

This workflow covers four stages:

1. **Generate** — Create a 3D model via Text-to-3D or Image-to-3D API  
2. **Optimize** — Remesh for print-ready topology and polycount  
3. **Export** — Convert to `.stl` or `.3mf`  
4. **Slice & Print** — Use slicers like Bambu Studio, Cura, etc.

---

## Prerequisites

- Python 3.8+
- Meshy API key  
- requests library:

```bash
pip install requests
