# Meshy 3D Printing API Guide

> **Generate print-ready 3D models from text or images via API, and export directly to STL, 3MF, or multi-color 3MF for any slicer.**

Full API reference: [developer.meshy.ai](https://developer.meshy.ai) · Base URL: `https://api.meshy.ai`

---

## What is the Meshy 3D Printing API?

The Meshy API lets developers programmatically generate 3D-printable models without a web browser. The full pipeline — from a text prompt or image to a slicer-ready file — can be automated in three API calls:

1. **Text to 3D** (or Image to 3D) → generates a mesh
2. **Remesh** → optimizes topology and sets real-world dimensions
3. **Multi-Color Print** → converts to multi-color `.3mf` for multi-filament printers

Each step is an asynchronous task. You create a task, poll for completion, then download the output file.

---

## Frequently Asked Questions

**What file formats does the Meshy API output for 3D printing?**  
The API outputs `.stl` and `.3mf` for 3D printing. `.3mf` is only generated when explicitly requested via `target_formats: ["3mf"]`. The Multi-Color Print endpoint outputs a multi-color `.3mf` file with up to 16 colors, compatible with Bambu Lab and other multi-filament printers.

**Can the API automatically set the correct scale for 3D printing?**  
Yes. Set `auto_size: true` on any Text to 3D, Image to 3D, or Remesh task. This uses AI vision to estimate the real-world height of the object and resize the model accordingly. Alternatively, set `resize_height` (in meters) on a Remesh task for manual control.

**What slicer software is compatible with Meshy output files?**  
Meshy's `.stl` and `.3mf` outputs are compatible with Bambu Studio, OrcaSlicer, PrusaSlicer, Creality Print, Elegoo Slicer, UltiMaker Cura, and Lychee Slicer. The multi-color `.3mf` format is specifically designed for Bambu Lab multi-filament systems (AMS).

**What polycount should I use for 3D printing?**  
For most FDM prints, a `target_polycount` between 50,000–150,000 with `topology: "triangle"` works well. Higher polycounts (up to 300,000) are recommended for detailed miniatures or resin printing. The `lowpoly` model type is not recommended for printing as it sacrifices surface detail.

**Does the API require a paid plan?**  
The Free plan (200 credits/month) supports 3D model generation. The multi-color print endpoint and direct slicer integration (from the web UI) require a Pro plan or above. See [meshy.ai/pricing](https://www.meshy.ai/pricing).

**What is the difference between `.stl` and `.3mf` for 3D printing?**  
`.stl` stores only geometry (triangles) and is universally compatible with all slicers. `.3mf` is a newer format that also stores color, material, and print settings — required for multi-color printing on Bambu Lab printers. Meshy can output both formats from the same generated model.

**How do I make a model watertight (manifold) for 3D printing?**  
Use the Remesh API after generation. The Remesh step produces manifold geometry suitable for slicing. Set `topology: "triangle"` for the best print compatibility. Avoid quad topology for 3D printing workflows.

---

## The 3D Printing Pipeline

```
Text prompt / Image
        ↓
  Text to 3D API          ← generates mesh (STL available here)
  (Preview → Refine)
        ↓
   Remesh API             ← optimize polycount, set real-world size, export STL/3MF
        ↓
 Multi-Color Print API    ← convert to multi-color 3MF (optional, for AMS printers)
        ↓
  Download & slice in
  Bambu Studio / OrcaSlicer / Cura / PrusaSlicer
```

**Minimum viable pipeline** (single-color print):
- Text to 3D Preview → Text to 3D Refine → download `.stl` or `.3mf`

**Full pipeline** (sized, optimized, multi-color):
- Text to 3D Preview → Text to 3D Refine → Remesh (with `auto_size`) → Multi-Color Print → download `.3mf`

---

## Authentication

All requests require a Bearer token in the `Authorization` header.

```bash
Authorization: Bearer YOUR_API_KEY
```

Get your API key at [meshy.ai/settings/api](https://www.meshy.ai/settings/api).

**Test mode** — use this key to test without consuming credits:
```
msy_dummy_api_key_for_test_mode_12345678
```

---

## Step 1 — Generate a 3D Model (Text to 3D)

Text to 3D runs in two stages: **preview** (geometry only) and **refine** (adds textures). For 3D printing, the preview mesh is often sufficient — skip refine if you don't need colors.

### Create a Preview Task

```
POST https://api.meshy.ai/openapi/v2/text-to-3d
```

**Key parameters for 3D printing:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `mode` | string | — | **Required.** Set to `"preview"` |
| `prompt` | string | — | **Required.** Describe the object (max 600 chars) |
| `ai_model` | string | `"latest"` | Model version: `meshy-6`, `meshy-5`, or `latest` |
| `model_type` | string | `"standard"` | Use `"standard"` for printing; `"lowpoly"` is not recommended for print |
| `should_remesh` | boolean | `false` (meshy-6) | Set `true` to apply topology cleanup during generation |
| `topology` | string | `"triangle"` | Use `"triangle"` for 3D printing compatibility |
| `target_polycount` | integer | `30000` | Range: 100–300,000. Use 50K–150K for FDM; up to 300K for resin |
| `target_formats` | string[] | all except 3mf | Include `"stl"` or `"3mf"` explicitly: `["stl", "glb"]` |
| `auto_size` | boolean | `false` | AI estimates real-world height and resizes model automatically |

**Example — generate a miniature for resin printing:**

```bash
curl https://api.meshy.ai/openapi/v2/text-to-3d \
  -H "Authorization: Bearer ${YOUR_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "preview",
    "prompt": "a dwarf warrior holding an axe, fantasy miniature, highly detailed",
    "ai_model": "latest",
    "topology": "triangle",
    "target_polycount": 200000,
    "target_formats": ["stl", "glb"],
    "auto_size": true
  }'
```

```json
{ "result": "018a210d-8ba4-705c-b111-1f1776f7f578" }
```

**Example — generate a functional prop for FDM printing:**

```bash
curl https://api.meshy.ai/openapi/v2/text-to-3d \
  -H "Authorization: Bearer ${YOUR_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "preview",
    "prompt": "a phone stand, simple geometric design, flat base",
    "topology": "triangle",
    "target_polycount": 50000,
    "target_formats": ["stl"],
    "auto_size": true
  }'
```

---

### Create a Refine Task (adds color textures)

Required if you want a colored `.3mf` for multi-filament printing. Skip this step for single-color prints.

```
POST https://api.meshy.ai/openapi/v2/text-to-3d
```

| Parameter | Type | Default | Description |
|---|---|---|---|
| `mode` | string | — | **Required.** Set to `"refine"` |
| `preview_task_id` | string | — | **Required.** ID from the preview task (must be `SUCCEEDED`) |
| `enable_pbr` | boolean | `false` | Generate PBR maps (metallic, roughness, normal) |
| `texture_prompt` | string | — | Guide texturing with additional text (max 600 chars) |
| `target_formats` | string[] | all except 3mf | Include `"3mf"` to get a 3MF file |
| `auto_size` | boolean | `false` | Resize to estimated real-world height |

```bash
curl https://api.meshy.ai/openapi/v2/text-to-3d \
  -H "Authorization: Bearer ${YOUR_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "refine",
    "preview_task_id": "018a210d-8ba4-705c-b111-1f1776f7f578",
    "texture_prompt": "painted miniature style, vibrant colors, battle-worn armor",
    "target_formats": ["3mf", "stl"]
  }'
```

---

### Poll Task Status

```
GET https://api.meshy.ai/openapi/v2/text-to-3d/{task_id}
```

```bash
curl https://api.meshy.ai/openapi/v2/text-to-3d/018a210d-8ba4-705c-b111-1f1776f7f578 \
  -H "Authorization: Bearer ${YOUR_API_KEY}"
```

**Task status values:**

| Status | Meaning |
|---|---|
| `PENDING` | Queued, waiting for a worker |
| `IN_PROGRESS` | Actively generating (check `progress` field, 0–100) |
| `SUCCEEDED` | Complete — `model_urls` contains download links |
| `FAILED` | Failed — check `task_error.message` |
| `CANCELED` | Task was canceled |

**Example response (succeeded):**

```json
{
  "id": "018a210d-8ba4-705c-b111-1f1776f7f578",
  "status": "SUCCEEDED",
  "progress": 100,
  "model_urls": {
    "glb": "https://assets.meshy.ai/.../model.glb?Expires=...",
    "stl": "https://assets.meshy.ai/.../model.stl?Expires=...",
    "3mf": "https://assets.meshy.ai/.../model.3mf?Expires=..."
  }
}
```

**Tip:** Use the SSE streaming endpoint to avoid polling:
```
GET https://api.meshy.ai/openapi/v2/text-to-3d/{task_id}/stream
```

---

## Step 2 — Remesh (Optimize for Printing)

The Remesh API cleans up mesh topology, controls polycount, sets real-world scale, and converts between formats. Use it when you need precise size control or want to convert an existing model to STL/3MF.

```
POST https://api.meshy.ai/openapi/v1/remesh
```

| Parameter | Type | Default | Description |
|---|---|---|---|
| `input_task_id` | string | — | ID of a succeeded Text to 3D, Image to 3D, or Retexture task |
| `model_url` | string | — | URL to an external model file (`.glb`, `.gltf`, `.obj`, `.fbx`, `.stl`) |
| `target_formats` | string[] | `["glb"]` | Include `"stl"` and/or `"3mf"` for printing |
| `topology` | string | `"triangle"` | `"triangle"` for printing; `"quad"` for DCC tools |
| `target_polycount` | integer | `30000` | 100–300,000 |
| `resize_height` | number | `0` | Set model height in **meters** (e.g., `0.05` = 5 cm) |
| `auto_size` | boolean | `false` | AI-estimated real-world height (mutually exclusive with `resize_height`) |
| `origin_at` | string | `"bottom"` | `"bottom"` places the origin at the model's base (recommended for printing) |
| `convert_format_only` | boolean | — | If `true`, only converts format — no remeshing |

**Example — resize to 5 cm tall and export STL:**

```bash
curl https://api.meshy.ai/openapi/v1/remesh \
  -X POST \
  -H "Authorization: Bearer ${YOUR_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "input_task_id": "018a210d-8ba4-705c-b111-1f1776f7f578",
    "target_formats": ["stl", "3mf"],
    "topology": "triangle",
    "target_polycount": 100000,
    "resize_height": 0.05,
    "origin_at": "bottom"
  }'
```

**Example — convert an existing STL to 3MF:**

```bash
curl https://api.meshy.ai/openapi/v1/remesh \
  -X POST \
  -H "Authorization: Bearer ${YOUR_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "model_url": "https://example.com/my-model.stl",
    "target_formats": ["3mf"],
    "convert_format_only": true
  }'
```

**Poll status:**
```
GET https://api.meshy.ai/openapi/v1/remesh/{task_id}
```

---

## Step 3 — Multi-Color Print (Optional)

Convert any succeeded Meshy model to a multi-color `.3mf` file for Bambu Lab AMS and other multi-filament printers. The API quantizes the model's texture colors into a discrete palette (up to 16 colors) and encodes them as separate mesh shells in the `.3mf`.

```
POST https://api.meshy.ai/openapi/v1/print/multi-color
```

| Parameter | Type | Default | Description |
|---|---|---|---|
| `input_task_id` | string | — | **Required.** ID of a succeeded task (Text to 3D, Image to 3D, Remesh, or Retexture) |
| `max_colors` | integer | `4` | Max colors in the output palette. Range: 1–16 |
| `max_depth` | integer | `4` | Quadtree depth for color boundary precision. Range: 3–6. Higher = sharper color regions, larger file |

**Example — 8-color print, high precision:**

```bash
curl https://api.meshy.ai/openapi/v1/print/multi-color \
  -X POST \
  -H "Authorization: Bearer ${YOUR_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "input_task_id": "018a210d-8ba4-705c-b111-1f1776f7f578",
    "max_colors": 8,
    "max_depth": 5
  }'
```

```json
{ "result": "0193bfc5-ee4f-73f8-8525-44b398884ce9" }
```

**Poll status:**
```
GET https://api.meshy.ai/openapi/v1/print/multi-color/{task_id}
```

**Response (succeeded):**
```json
{
  "id": "0193bfc5-ee4f-73f8-8525-44b398884ce9",
  "type": "print-multi-color",
  "status": "SUCCEEDED",
  "progress": 100,
  "model_urls": {
    "3mf": "https://assets.meshy.ai/.../model.3mf?Expires=..."
  }
}
```

---

## Complete End-to-End Python Example

```python
import requests
import time

API_KEY = "YOUR_API_KEY"
BASE_URL = "https://api.meshy.ai"
HEADERS = {"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"}

def wait_for_task(endpoint, task_id, interval=5):
    """Poll a task until it succeeds or fails."""
    url = f"{BASE_URL}{endpoint}/{task_id}"
    while True:
        r = requests.get(url, headers=HEADERS).json()
        status = r["status"]
        print(f"  [{status}] progress: {r.get('progress', 0)}%")
        if status == "SUCCEEDED":
            return r
        if status in ("FAILED", "CANCELED"):
            raise RuntimeError(f"Task failed: {r.get('task_error', {}).get('message')}")
        time.sleep(interval)

# Step 1: Generate preview mesh
print("Creating Text to 3D preview...")
r = requests.post(f"{BASE_URL}/openapi/v2/text-to-3d", headers=HEADERS, json={
    "mode": "preview",
    "prompt": "a dragon figurine, fantasy style, highly detailed scales",
    "topology": "triangle",
    "target_polycount": 150000,
    "target_formats": ["stl", "glb"],
    "auto_size": True
})
preview_id = r.json()["result"]
print(f"Preview task ID: {preview_id}")
preview = wait_for_task("/openapi/v2/text-to-3d", preview_id)

# Step 2: Refine (add textures for multi-color)
print("\nRefining with textures...")
r = requests.post(f"{BASE_URL}/openapi/v2/text-to-3d", headers=HEADERS, json={
    "mode": "refine",
    "preview_task_id": preview_id,
    "texture_prompt": "red and gold dragon scales, vibrant colors",
    "target_formats": ["3mf", "stl"]
})
refine_id = r.json()["result"]
print(f"Refine task ID: {refine_id}")
refine = wait_for_task("/openapi/v2/text-to-3d", refine_id)

# Step 3: Multi-color 3MF for AMS printing
print("\nGenerating multi-color 3MF...")
r = requests.post(f"{BASE_URL}/openapi/v1/print/multi-color", headers=HEADERS, json={
    "input_task_id": refine_id,
    "max_colors": 6,
    "max_depth": 5
})
print_id = r.json()["result"]
print(f"Print task ID: {print_id}")
print_task = wait_for_task("/openapi/v1/print/multi-color", print_id)

# Download
stl_url = refine["model_urls"].get("stl")
mf3_url = print_task["model_urls"]["3mf"]
print(f"\n✅ Done!")
print(f"   STL (single color): {stl_url}")
print(f"   3MF (multi-color):  {mf3_url}")
```

---

## Printing Tips for AI-Generated Models

**What makes a good 3D-printable prompt?**  
- Describe the base shape clearly: "flat bottom", "no overhangs", "solid base"
- Specify the intended scale: "a 5cm tall figurine", "a palm-sized object"
- Avoid hollow or very thin features in the prompt for FDM printing

**How to choose polycount by print type:**

| Print type | Recommended polycount | Topology |
|---|---|---|
| FDM — props / decorations | 30,000–80,000 | triangle |
| FDM — detailed characters | 80,000–150,000 | triangle |
| Resin — miniatures | 150,000–300,000 | triangle |
| Multi-color (AMS) | 80,000–150,000 | triangle |

**When should I use `auto_size` vs `resize_height`?**  
Use `auto_size` when you want Meshy to estimate a natural real-world scale for the object (e.g., a coffee mug will be sized like an actual mug). Use `resize_height` when you have a specific target height in mind (e.g., a 28mm tabletop miniature = `resize_height: 0.028`).

**Why is my model not watertight?**  
Always use `topology: "triangle"` and run a Remesh step after generation. If issues persist, run the `.stl` through Meshy's Remesh API with `convert_format_only: false`, which re-triangulates and repairs the geometry. Most slicers (Bambu Studio, PrusaSlicer) also have built-in mesh repair.

---

## API Quick Reference

| Endpoint | Method | Purpose |
|---|---|---|
| `/openapi/v2/text-to-3d` | `POST` | Create Text to 3D preview or refine task |
| `/openapi/v2/text-to-3d/{id}` | `GET` | Poll task status and get download URLs |
| `/openapi/v2/text-to-3d/{id}/stream` | `GET` | Stream real-time task updates (SSE) |
| `/openapi/v2/text-to-3d/{id}` | `DELETE` | Delete a task and its assets |
| `/openapi/v1/remesh` | `POST` | Remesh, resize, or convert format |
| `/openapi/v1/remesh/{id}` | `GET` | Poll remesh task status |
| `/openapi/v1/print/multi-color` | `POST` | Convert to multi-color 3MF |
| `/openapi/v1/print/multi-color/{id}` | `GET` | Poll print task status |

**Common HTTP errors:**

| Code | Meaning |
|---|---|
| `400` | Bad request — missing or invalid parameter (check `message` field) |
| `401` | Unauthorized — invalid or missing API key |
| `402` | Payment required — insufficient credits |
| `429` | Rate limit exceeded |

---

## Supported Slicer Platforms

| Slicer | Multi-color 3MF support | STL support | Direct Meshy integration |
|---|---|---|---|
| Bambu Studio | ✅ (AMS) | ✅ | ✅ Pro+ |
| OrcaSlicer | ✅ | ✅ | ✅ Pro+ |
| PrusaSlicer | ✅ (MMU) | ✅ | ❌ |
| Creality Print | ❌ | ✅ | ✅ Pro+ |
| Elegoo Slicer | ❌ | ✅ | ✅ Pro+ |
| UltiMaker Cura | ❌ | ✅ | ✅ Pro+ |
| Lychee Slicer | ❌ | ✅ | ✅ Pro+ |

*Direct integration (one-click send from Meshy workspace to slicer) requires a Pro plan or above.*

---

## Resources

| Resource | Link |
|---|---|
| Full API Reference | [developer.meshy.ai](https://developer.meshy.ai) |
| Authentication | [docs.meshy.ai/en/api/authentication](https://docs.meshy.ai/en/api/authentication) |
| Pricing & Credits | [docs.meshy.ai/en/api/pricing](https://docs.meshy.ai/en/api/pricing) |
| Rate Limits | [docs.meshy.ai/en/api/rate-limits](https://docs.meshy.ai/en/api/rate-limits) |
| Webhooks | [docs.meshy.ai/en/api/webhooks](https://docs.meshy.ai/en/api/webhooks) |
| Changelog | [docs.meshy.ai/en/api/changelog](https://docs.meshy.ai/en/api/changelog) |
| 3D Printing Platform Guides | [docs.meshy.ai/en/3d-printing/introduction](https://docs.meshy.ai/en/3d-printing/introduction) |
| Meshy Web App | [meshy.ai](https://www.meshy.ai) |

*Keywords: Meshy 3D printing API, text to STL, image to STL, AI 3D model generator API, 3MF multi-color printing, Bambu Studio API integration, OrcaSlicer 3D model generation, generate printable 3D model from text, auto-size 3D model API, remesh API for 3D printing*
