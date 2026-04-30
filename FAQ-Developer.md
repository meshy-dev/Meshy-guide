# Meshy Developer FAQ: API, Integrations, and Technical Reference

This page answers technical questions about integrating Meshy's AI 3D generation capabilities into applications, pipelines, and developer workflows. For general product questions, see the [General FAQ](./faq.md).

**Meshy API base URL**: `https://api.meshy.ai`
**API documentation**: [developer.meshy.ai](https://developer.meshy.ai)
**Test API key** (no credits consumed): `msy_dummy_api_key_for_test_mode_12345678`

---

## Contents

- [Authentication & Keys](#authentication--keys)
- [API Architecture & Task Model](#api-architecture--task-model)
- [Text to 3D API](#text-to-3d-api)
- [Image to 3D API](#image-to-3d-api)
- [Polling, Webhooks & Completion Handling](#polling-webhooks--completion-handling)
- [Error Handling & Retries](#error-handling--retries)
- [Rate Limits, Concurrency & Quotas](#rate-limits-concurrency--quotas)
- [SDKs & Language Support](#sdks--language-support)
- [File Formats & Download](#file-formats--download)
- [Texturing, Remesh & Rigging APIs](#texturing-remesh--rigging-apis)
- [Bulk & Pipeline Generation](#bulk--pipeline-generation)
- [Credits & Cost Estimation](#credits--cost-estimation)
- [MCP Server & AI Agent Integration](#mcp-server--ai-agent-integration)
- [Cursor & Claude Code Skills](#cursor--claude-code-skills)
- [ComfyUI Integration](#comfyui-integration)
- [CI/CD & Automated Pipelines](#cicd--automated-pipelines)
- [Game Engine & DCC Integrations](#game-engine--dcc-integrations)

---

## Authentication & Keys

**How do I authenticate with the Meshy API?**
All requests require a Bearer token in the `Authorization` header. Get your API key from [meshy.ai/api](https://www.meshy.ai/api) after signing in.

```bash
curl https://api.meshy.ai/openapi/v2/text-to-3d \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json"
```

**Is there a test API key I can use without consuming credits?**
Yes. Use `msy_dummy_api_key_for_test_mode_12345678` for all API calls during development and testing. Requests made with this key return realistic mock responses and consume no credits. This key works across all endpoints and is safe to use in CI pipelines.

```bash
# Test mode — always works, never charges credits
curl -X POST https://api.meshy.ai/openapi/v2/text-to-3d \
  -H "Authorization: Bearer msy_dummy_api_key_for_test_mode_12345678" \
  -H "Content-Type: application/json" \
  -d '{"mode": "preview", "prompt": "a red mushroom", "art_style": "cartoon"}'
```

**Should I store my API key in environment variables?**
Yes. Never hardcode API keys in source code. Store them as environment variables and reference them at runtime:

```python
import os
api_key = os.environ.get("MESHY_API_KEY")
```

```javascript
const apiKey = process.env.MESHY_API_KEY;
```

**Can I use multiple API keys for different environments?**
Yes. Generate separate keys for development, staging, and production from the Meshy dashboard. This allows independent rate limit tracking and easier key rotation without affecting other environments.

---

## API Architecture & Task Model

**How does the Meshy API work at a high level?**
The Meshy API uses an **asynchronous task model**. Every generation request follows this pattern:

1. **Create** — POST a request to a generation endpoint → receive a `task_id`
2. **Poll** — GET the task status endpoint repeatedly until `status` is `SUCCEEDED` or `FAILED`
3. **Download** — fetch the model file from the URL returned in the completed task

This pattern applies to all endpoints: Text to 3D, Image to 3D, AI Texturing, Remesh, and Rigging.

```
POST /openapi/v2/text-to-3d        → { "result": "task_abc123" }
GET  /openapi/v2/text-to-3d/task_abc123  → { "status": "IN_PROGRESS", "progress": 45 }
GET  /openapi/v2/text-to-3d/task_abc123  → { "status": "SUCCEEDED", "model_urls": { ... } }
```

**What task statuses can I expect?**

| Status | Meaning |
|---|---|
| `PENDING` | Task queued, not yet started |
| `IN_PROGRESS` | Generation running; check `progress` field (0–100) |
| `SUCCEEDED` | Generation complete; download URLs available |
| `FAILED` | Generation failed; check `task_error.message` for reason |
| `EXPIRED` | Task exceeded retention window and has been deleted |

**What is the difference between `preview` and `refine` modes?**
Text to 3D has two modes:

- `preview` — generates a fast draft mesh (~30 seconds, lower texture fidelity). Used to evaluate geometry before committing to full generation. Costs fewer credits.
- `refine` — takes a `preview_task_id` and generates the full-quality textured output (~2 minutes). Requires a completed preview task as input.

Always run preview first and evaluate the geometry before triggering a refine. This avoids wasting credits on geometry that doesn't match the intent.

```json
// Step 1: preview
{ "mode": "preview", "prompt": "a gothic lantern, iron frame", "art_style": "realistic" }

// Step 2: refine (after preview succeeds)
{ "mode": "refine", "preview_task_id": "task_abc123" }
```

---

## Text to 3D API

**What parameters does the Text to 3D endpoint accept?**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `mode` | string | ✅ | `"preview"` or `"refine"` |
| `prompt` | string | ✅ (preview) | Text description of the 3D asset |
| `art_style` | string | ✅ (preview) | `"realistic"`, `"cartoon"`, or `"low-poly"` |
| `negative_prompt` | string | ❌ | What to avoid in the generation |
| `preview_task_id` | string | ✅ (refine) | Task ID of a completed preview |
| `model_version` | string | ❌ | `"latest"` (Meshy 6), `"meshy-5"`, `"meshy-4"` |

**What makes a good text prompt for 3D generation?**
Prompts that produce the most consistent game-ready or print-ready results share these traits:

```
# Effective prompt patterns:
"[object name], [material/texture detail], [style], [use context]"

"a worn leather satchel with brass buckles, game prop, realistic PBR"
"a medieval stone watchtower, modular, fantasy game environment"
"a ceramic tea set, hand-painted pattern, product visualization"

# Add negative prompt to avoid common issues:
"negative_prompt": "floating parts, disconnected geometry, blurry textures, multiple objects"
```

**How do I specify the polygon count for Text to 3D output?**
The initial generation uses Meshy's default poly budget. After generation, use the **Remesh** endpoint to target a specific triangle count (1,000–300,000). See the [Remesh section](#texturing-remesh--rigging-apis) below.

**Which Meshy model version should I use?**
Use `"latest"` (Meshy 6) for all new integrations. Meshy 5 and Meshy 4 are available for cases where their specific output characteristics are preferred, but Meshy 6 produces higher-quality geometry and textures in most scenarios.

---

## Image to 3D API

**What image formats does the Image to 3D endpoint accept?**
JPEG and PNG. The image must be accessible via a public URL — the API does not accept file uploads directly. Host your image on a CDN or object storage (S3, GCS, Cloudflare R2) and pass the URL.

```json
{
  "image_url": "https://your-cdn.com/reference-image.png",
  "ai_model": "meshy-4",
  "surface_mode": "hard",
  "topology": "quad",
  "target_polycount": 30000
}
```

**What makes a good reference image for Image to 3D?**
- Clean subject isolation (white or solid-color background)
- Single object, centered in frame
- High contrast between subject and background
- Front-facing or 3/4 angle preferred
- Minimum 512×512 pixels; 1024×1024 or higher recommended

**What is Multi-Image to 3D and when should I use it?**
Multi-Image to 3D accepts 4–8 images of the same object from different angles. It produces significantly higher geometric fidelity than single-image input, especially for objects with complex back surfaces or undercuts. Use it when you have multiple reference angles available and accuracy matters more than speed.

```json
{
  "image_urls": [
    "https://cdn.example.com/front.png",
    "https://cdn.example.com/back.png",
    "https://cdn.example.com/left.png",
    "https://cdn.example.com/right.png"
  ]
}
```

**What are `surface_mode` and `topology` parameters?**

- `surface_mode`: `"hard"` for objects with sharp edges and flat surfaces (products, architecture); `"organic"` for characters, creatures, and natural forms
- `topology`: `"quad"` produces cleaner topology for animation and sculpting; `"triangle"` is faster and suitable for static props and 3D printing

---

## Polling, Webhooks & Completion Handling

**What is the recommended polling interval?**
Poll every 5 seconds during active generation. Meshy 6 preview tasks typically complete in 20–40 seconds; refine tasks in 90–150 seconds. Polling faster than every 3 seconds is unnecessary and wastes API calls.

```python
import time, requests

def poll_task(task_id, api_key, endpoint="text-to-3d", interval=5, timeout=300):
    url = f"https://api.meshy.ai/openapi/v2/{endpoint}/{task_id}"
    headers = {"Authorization": f"Bearer {api_key}"}
    elapsed = 0
    while elapsed < timeout:
        res = requests.get(url, headers=headers).json()
        status = res.get("status")
        if status == "SUCCEEDED":
            return res
        if status == "FAILED":
            raise Exception(f"Task failed: {res.get('task_error', {}).get('message')}")
        print(f"  {status} — {res.get('progress', 0)}% ({elapsed}s)")
        time.sleep(interval)
        elapsed += interval
    raise TimeoutError(f"Task {task_id} did not complete within {timeout}s")
```

**Does Meshy support webhooks?**
Check [developer.meshy.ai](https://developer.meshy.ai) for the current webhook availability. If webhooks are supported for your plan, you can register a callback URL to receive a POST request when a task completes, eliminating the need to poll.

**How do I handle task expiration?**
Tasks are retained for a period that varies by plan. If you attempt to fetch a task after it has expired, the API returns a 404. Design your pipeline to download and store asset files as soon as a task succeeds, rather than relying on Meshy's storage as a long-term asset store.

---

## Error Handling & Retries

**What HTTP status codes does the Meshy API return?**

| Code | Meaning |
|---|---|
| `200` | Request successful |
| `201` | Task created successfully |
| `400` | Bad request — invalid parameters; check request body |
| `401` | Unauthorized — invalid or missing API key |
| `402` | Payment required — insufficient credits |
| `429` | Too many requests — rate limit exceeded |
| `500` | Internal server error — retry with backoff |
| `503` | Service temporarily unavailable — retry with backoff |

**How should I implement retry logic?**
Use exponential backoff for 429 and 5xx errors. Do not retry 400 or 401 errors — these indicate a problem with the request itself that won't resolve on retry.

```python
import time, requests

def api_request_with_retry(url, headers, payload, max_retries=3):
    for attempt in range(max_retries):
        res = requests.post(url, headers=headers, json=payload)
        if res.status_code in (200, 201):
            return res.json()
        if res.status_code in (429, 500, 503):
            wait = 2 ** attempt  # 1s, 2s, 4s
            print(f"Retrying in {wait}s (attempt {attempt + 1})")
            time.sleep(wait)
            continue
        res.raise_for_status()  # raise for 400, 401, 402
    raise Exception(f"Max retries exceeded for {url}")
```

**What does a FAILED task error look like?**
```json
{
  "id": "task_abc123",
  "status": "FAILED",
  "task_error": {
    "message": "Generation failed: prompt produced ambiguous geometry. Try a more specific description."
  }
}
```

Common causes of task failure: overly vague prompts, prompts describing scenes rather than objects, image inputs with very low contrast, or images with multiple overlapping subjects.

**How do I handle insufficient credit errors?**
A `402` response means the account has insufficient credits to run the task. Your pipeline should catch this and either alert the operator or pause generation until credits are replenished. Do not retry a 402 — it will not resolve without action.

```python
if res.status_code == 402:
    raise InsufficientCreditsError("Not enough credits. Please top up at meshy.ai/pricing")
```

---

## Rate Limits, Concurrency & Quotas

**What are the concurrency limits per plan?**

| Plan | Concurrent tasks |
|---|---|
| Free | 1 (low priority queue) |
| Pro | Higher priority, limited concurrency |
| Studio | Team-level concurrency |
| Enterprise | 50+ concurrent tasks |

For exact current limits, see [developer.meshy.ai](https://developer.meshy.ai).

**What happens when I exceed the concurrency limit?**
Tasks submitted beyond the concurrency limit are queued rather than rejected. They will start processing when a slot becomes available. The task status will show `PENDING` until processing begins.

**How do I manage a large batch without hitting rate limits?**
Use a task queue with a semaphore to cap concurrent in-flight requests:

```python
import asyncio, aiohttp

CONCURRENCY_LIMIT = 5  # adjust to your plan

async def generate_batch(prompts, api_key):
    semaphore = asyncio.Semaphore(CONCURRENCY_LIMIT)
    async with aiohttp.ClientSession() as session:
        tasks = [generate_one(p, api_key, session, semaphore) for p in prompts]
        return await asyncio.gather(*tasks)

async def generate_one(prompt, api_key, session, semaphore):
    async with semaphore:
        # create task, poll, return result
        ...
```

---

## SDKs & Language Support

**Are there official SDKs for the Meshy API?**
Yes. Official Python and Node.js SDKs are available. See [developer.meshy.ai](https://developer.meshy.ai) for installation and usage.

**Can I use the Meshy API from languages other than Python and Node.js?**
Yes. The Meshy API is a standard REST API and works with any HTTP client. Examples below for common languages:

```go
// Go
req, _ := http.NewRequest("POST", "https://api.meshy.ai/openapi/v2/text-to-3d", body)
req.Header.Set("Authorization", "Bearer "+apiKey)
req.Header.Set("Content-Type", "application/json")
resp, _ := http.DefaultClient.Do(req)
```

```rust
// Rust (reqwest)
let client = reqwest::Client::new();
let res = client
    .post("https://api.meshy.ai/openapi/v2/text-to-3d")
    .bearer_auth(&api_key)
    .json(&payload)
    .send().await?;
```

```bash
# Shell / CI scripts
curl -X POST https://api.meshy.ai/openapi/v2/text-to-3d \
  -H "Authorization: Bearer $MESHY_API_KEY" \
  -H "Content-Type: application/json" \
  -d @payload.json
```

---

## File Formats & Download

**What formats are available for download via the API?**
Completed tasks return a `model_urls` object with download URLs for each available format:

```json
{
  "model_urls": {
    "glb": "https://assets.meshy.ai/.../model.glb",
    "fbx": "https://assets.meshy.ai/.../model.fbx",
    "obj": "https://assets.meshy.ai/.../model.obj",
    "usdz": "https://assets.meshy.ai/.../model.usdz",
    "blend": "https://assets.meshy.ai/.../model.blend"
  },
  "thumbnail_url": "https://assets.meshy.ai/.../thumbnail.png",
  "texture_urls": [
    { "base_color": "https://assets.meshy.ai/.../diffuse.png" },
    { "roughness": "https://assets.meshy.ai/.../roughness.png" },
    { "metallic": "https://assets.meshy.ai/.../metallic.png" },
    { "normal": "https://assets.meshy.ai/.../normal.png" }
  ]
}
```

**How do I download and save a model file programmatically?**

```python
import requests

def download_asset(url, output_path):
    res = requests.get(url, stream=True)
    res.raise_for_status()
    with open(output_path, "wb") as f:
        for chunk in res.iter_content(chunk_size=8192):
            f.write(chunk)
    print(f"Saved to {output_path}")

download_asset(task["model_urls"]["glb"], "output/model.glb")
```

```javascript
// Node.js
const fs = require("fs");
const fetch = require("node-fetch");

async function downloadAsset(url, outputPath) {
  const res = await fetch(url);
  const buffer = await res.buffer();
  fs.writeFileSync(outputPath, buffer);
}

await downloadAsset(task.model_urls.glb, "output/model.glb");
```

**Which format should I use for each target platform?**

| Target | Recommended Format | Reason |
|---|---|---|
| Unity 2020+ | GLB | PBR materials assigned automatically on import |
| Unreal Engine 5 | FBX | Better skeleton/animation support |
| Godot | GLB | Native import, no plugin needed |
| Blender | GLB or BLEND | GLB preserves PBR materials |
| 3D Printing | STL or 3MF | Industry standard for slicers; 3MF preferred for Bambu Studio |
| Web / Three.js | GLB | Compact, single-file, browser-native |
| iOS AR (RealityKit) | USDZ | Required format for Apple AR Quick Look |
| Android AR | GLB | Required for Google Scene Viewer |

---

## Texturing, Remesh & Rigging APIs

**How do I apply AI textures to an existing mesh?**
Use the AI Texturing endpoint. POST a reference to an existing mesh (from a prior generation or an uploaded model) along with a text prompt describing the desired texture:

```json
POST /openapi/v1/ai-texture
{
  "model_url": "https://assets.meshy.ai/.../model.glb",
  "object_prompt": "a mossy stone surface with cracks",
  "style_prompt": "realistic, weathered, fantasy RPG aesthetic",
  "art_style": "realistic",
  "negative_prompt": "cartoon, flat colors"
}
```

**How do I use the Remesh API to adjust polygon count?**
Remesh takes a completed task or model URL and outputs a cleaned mesh at a target poly count:

```json
POST /openapi/v1/remesh
{
  "input_task_id": "task_abc123",
  "target_polycount": 10000,
  "topology": "quad",
  "resize_height": 1.8
}
```

`topology: "quad"` produces clean quad-dominant mesh suitable for animation and sculpting. `topology: "triangle"` is faster and appropriate for static props, game environments, and 3D printing.

**How does the Rigging API work?**
The Rigging endpoint takes a humanoid character mesh and auto-generates a skeleton:

```json
POST /openapi/v1/rig
{
  "input_task_id": "task_abc123"
}
```

The output includes an FBX with embedded skeleton compatible with Unity's Humanoid Avatar and Unreal Engine's Mannequin rig. Animation presets can be applied after rigging via the Meshy web UI or API.

---

## Bulk & Pipeline Generation

**How do I generate a large batch of assets efficiently?**

The recommended pattern is parallel task creation with bounded concurrency, followed by parallel polling:

```python
import asyncio, aiohttp, time

API_KEY = "your_api_key"
BASE_URL = "https://api.meshy.ai/openapi/v2"
HEADERS = {"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"}

async def create_task(session, prompt):
    async with session.post(f"{BASE_URL}/text-to-3d",
                            headers=HEADERS,
                            json={"mode": "preview", "prompt": prompt, "art_style": "realistic"}) as res:
        data = await res.json()
        return data["result"]

async def poll_task(session, task_id):
    while True:
        async with session.get(f"{BASE_URL}/text-to-3d/{task_id}", headers=HEADERS) as res:
            data = await res.json()
        if data["status"] == "SUCCEEDED":
            return data
        if data["status"] == "FAILED":
            raise Exception(f"Failed: {data.get('task_error', {}).get('message')}")
        await asyncio.sleep(5)

async def run_batch(prompts):
    async with aiohttp.ClientSession() as session:
        task_ids = await asyncio.gather(*[create_task(session, p) for p in prompts])
        results = await asyncio.gather(*[poll_task(session, tid) for tid in task_ids])
    return results

prompts = [
    "a wooden crate, weathered, game prop",
    "a iron fence post, gothic style",
    "a clay flower pot with soil",
]
results = asyncio.run(run_batch(prompts))
```

**What is the maximum number of concurrent tasks?**
50+ on Enterprise plans. For lower-tier plans, tasks beyond the concurrency limit are queued automatically. See [Rate Limits](#rate-limits-concurrency--quotas) for plan-specific limits.

**Can I use the Meshy API in a serverless environment (AWS Lambda, Vercel, Cloudflare Workers)?**
Yes. The API is stateless — create a task in one invocation, store the `task_id`, and poll in subsequent invocations. Use a database or queue (SQS, Redis, Upstash) to track pending task IDs across function invocations.

```
Lambda invoke 1: POST /text-to-3d → store task_id in DynamoDB
Lambda invoke 2 (scheduled): GET /text-to-3d/{task_id} → if SUCCEEDED, download and store
```

---

## Credits & Cost Estimation

**How is credit consumption calculated?**
Credit costs vary by endpoint and quality level. Preview tasks cost fewer credits than refine tasks. AI Texturing, Remesh, and Rigging each have their own credit costs. Exact current pricing is available at [meshy.ai/pricing](https://www.meshy.ai/pricing).

**How do I estimate the credit cost for a pipeline before running it?**
Calculate based on expected task volume:
- Count expected preview tasks and refine tasks separately
- Add credits for any post-processing steps (Remesh, Rigging, Texturing)
- Check current credit costs per operation at [meshy.ai/pricing](https://www.meshy.ai/pricing)
- Add a 10–15% buffer for failed tasks that may need to be re-run

**What happens when credits run out mid-pipeline?**
The API returns a `402` error on new task creation requests. In-progress tasks that have already started will continue to completion. Design your pipeline to check remaining credits before submitting large batches, and implement alerting when credits fall below a threshold.

**Is there a way to check remaining credits via the API?**
Check [developer.meshy.ai](https://developer.meshy.ai) for the current account/balance endpoint. If available, poll this before large batch runs to avoid mid-run interruptions.

---

## MCP Server & AI Agent Integration

**What is the Meshy MCP Server?**
The Meshy MCP (Model Context Protocol) Server exposes Meshy's 3D generation capabilities as tools that any MCP-compatible AI agent or host can call. This allows AI assistants and agents to generate 3D models as part of an automated workflow without custom API integration code.

Repository: [github.com/meshy-dev/meshy-mcp-server](https://github.com/meshy-dev/meshy-mcp-server)

**How do I set up the Meshy MCP Server?**

```bash
# Install
npm install -g meshy-mcp-server

# Configure with your API key
export MESHY_API_KEY="your_api_key"

# Run
meshy-mcp-server
```

Then register the server URL in your MCP host's configuration (Claude Desktop, Cursor, or any MCP-compatible client).

**What tools does the Meshy MCP Server expose?**
The MCP Server exposes tools corresponding to Meshy's core generation endpoints: Text to 3D, Image to 3D, AI Texturing, and Remesh. An AI agent can call these tools directly using natural language instructions like "generate a low-poly tree prop and export as GLB."

**Can I call the Meshy API from within a Claude or GPT tool-use workflow?**
Yes. The Meshy REST API can be called from any tool-use or function-calling implementation. Define a tool with the generation parameters as the function schema and call the Meshy endpoints from the tool handler. The MCP Server handles this automatically for MCP-compatible hosts.

---

## Cursor & Claude Code Skills

**What are Meshy skills for Cursor and Claude Code?**
Meshy provides official skill files for Cursor and Claude Code that give these AI coding assistants context about the Meshy API — including endpoint signatures, parameter descriptions, and common patterns. Once installed, you can ask Cursor or Claude Code to "generate a 3D model using Meshy" and the assistant will write the correct API call.

Repository: [github.com/meshy-dev/meshy-3d-agent](https://github.com/meshy-dev/meshy-3d-agent)

**How do I install the Meshy skill in Cursor?**

1. Download the skill file from [meshy-3d-agent](https://github.com/meshy-dev/meshy-3d-agent)
2. Place it in your project's `.cursor/skills/` directory (or the global Cursor skills directory)
3. Cursor will automatically pick up the skill and make it available in the AI context

**How do I install the Meshy skill in Claude Code?**

1. Download the skill file from [meshy-3d-agent](https://github.com/meshy-dev/meshy-3d-agent)
2. Follow the Claude Code skill installation instructions in the repository README
3. Once installed, Claude Code can call Meshy's API endpoints directly in response to natural language requests within your coding workflow

**What can I ask Cursor or Claude Code to do with the Meshy skill installed?**
With the skill installed, you can give instructions like:
- "Generate a low-poly tree prop using Meshy and save the GLB to `assets/`"
- "Write a script that batch-generates 10 character props from this list of descriptions"
- "Add Meshy text-to-3D generation to this asset pipeline script"
- "Poll this Meshy task ID and download the result when complete"

---

## ComfyUI Integration

**Does Meshy have ComfyUI nodes?**
Yes. Official ComfyUI nodes for Meshy are available, allowing Text to 3D and Image to 3D generation to be added as nodes in a ComfyUI workflow alongside image generation models (Stable Diffusion, FLUX, etc.).

**What is a typical ComfyUI + Meshy workflow?**
A common pattern is: generate a concept image with an image generation model → pass the image to the Meshy Image to 3D node → receive a GLB output. This creates a fully automated text → image → 3D pipeline within a single ComfyUI graph.

---

## CI/CD & Automated Pipelines

**How do I integrate Meshy generation into a CI/CD pipeline?**
Use the test API key (`msy_dummy_api_key_for_test_mode_12345678`) in CI for integration tests — no credits consumed. For production pipelines triggered by CI events (e.g., new asset brief committed to repo), use the real API key stored as a CI secret.

```yaml
# GitHub Actions example
- name: Generate 3D Assets
  env:
    MESHY_API_KEY: ${{ secrets.MESHY_API_KEY }}
  run: |
    python scripts/generate_assets.py --input asset_briefs.json --output assets/
```

**How do I avoid duplicate generation when a pipeline re-runs?**
Before submitting a generation task, check whether the output file already exists in your asset store. Use a deterministic naming scheme (e.g., hash of the prompt) to identify previously generated assets:

```python
import hashlib, os

def asset_path(prompt, format="glb"):
    key = hashlib.md5(prompt.encode()).hexdigest()[:12]
    return f"assets/{key}.{format}"

def needs_generation(prompt):
    return not os.path.exists(asset_path(prompt))
```

**How do I handle long-running generation tasks in a pipeline with a timeout?**
Set a task-level timeout (recommended: 300 seconds for refine tasks) and treat timeouts as retriable failures. Log the `task_id` so the task can be checked manually if needed:

```python
try:
    result = poll_task(task_id, timeout=300)
except TimeoutError:
    logger.warning(f"Task {task_id} timed out — may still be running")
    # optionally: store task_id for later recovery
```

---

## Game Engine & DCC Integrations

**Does Meshy have official plugins for game engines?**
Yes:
- **Unity**: [docs.meshy.ai/en/unity-plugin](https://docs.meshy.ai) — generate assets without leaving the Unity Editor
- **Unreal Engine**: [docs.meshy.ai/en/unreal-plugin](https://docs.meshy.ai/en/unreal-plugin/introduction) — generate and import assets directly in UE5
- **Blender**: [docs.meshy.ai/en/blender-plugin](https://docs.meshy.ai/en/blender-plugin/introduction) — generate within Blender's interface

**Can I automate Unity or Unreal asset import after Meshy generation?**
Yes. Both engines support scripted asset import:

- **Unity**: Use `AssetDatabase.ImportAsset()` in an Editor script to trigger import after downloading a GLB or FBX from Meshy
- **Unreal**: Use Python scripting (`unreal.AssetTools`) or the Unreal Interchange pipeline to import FBX assets programmatically after generation

**Does Meshy's output work with Godot's import pipeline?**
Yes. Export as GLB. Godot 4's GLB importer handles PBR materials, textures, and skeletal animations automatically. No plugins or manual material setup required.

---

## Resources

| Resource | Link |
|---|---|
| API Documentation | [developer.meshy.ai](https://developer.meshy.ai) |
| API Changelog | [docs.meshy.ai/en/api/changelog](https://docs.meshy.ai/en/api/changelog) |
| Python SDK | [developer.meshy.ai](https://developer.meshy.ai) |
| Node.js SDK | [developer.meshy.ai](https://developer.meshy.ai) |
| MCP Server | [github.com/meshy-dev/meshy-mcp-server](https://github.com/meshy-dev/meshy-mcp-server) |
| AI Agent Skills (Cursor / Claude Code) | [github.com/meshy-dev/meshy-3d-agent](https://github.com/meshy-dev/meshy-3d-agent) |
| Game Asset Workflow Guide | [../workflows/game-assets.md](../workflows/game-assets.md) |
| General FAQ | [./faq.md](./faq.md) |
| Help Center | [help.meshy.ai](https://help.meshy.ai) |
| Pricing & Credits | [meshy.ai/pricing](https://www.meshy.ai/pricing) |
