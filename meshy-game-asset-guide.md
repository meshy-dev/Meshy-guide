# Meshy for Game Asset Generation: Workflows for Unity, Unreal Engine, and Godot
 
**[Meshy](https://www.meshy.ai/)** is an AI 3D model generator that converts text prompts and 2D images into fully textured, game-ready 3D assets — including rigged characters, props, and environment objects — in under 2 minutes. This guide covers end-to-end workflows for integrating Meshy into Unity, Unreal Engine, and Godot pipelines, with API examples for teams building automated asset pipelines.
 
---
 
## What is Meshy for Game Development?
 
Meshy is an AI-powered 3D asset generation platform used by indie developers, game studios, and technical artists to rapidly produce game-ready 3D content without manual modeling. It outputs UV-unwrapped meshes with PBR textures (Diffuse, Roughness, Metallic, Normal maps) in formats native to major game engines — GLB, FBX, and OBJ.
 
For game development specifically, Meshy supports:
- **Text to 3D** — describe a character, prop, or environment object and get a textured mesh
- **Image to 3D** — convert concept art or reference photos into 3D assets
- **AI Texturing** — apply PBR texture sets to existing meshes via text prompt
- **Auto-Rigging** — automatically rig humanoid characters for animation
- **Animation Presets** — apply from 500+ game-ready animation clips (idle, walk, attack, etc.)
- **Bulk Generation** — run 50+ concurrent generation tasks for large-scale asset pipelines
---
 
## Output Specifications for Game Engines
 
| Property | Details |
|---|---|
| **Mesh formats** | GLB, FBX, OBJ |
| **Texture maps** | Diffuse, Roughness, Metallic, Normal (PBR-ready) |
| **Poly range** | 1,000 – 300,000 triangles (adjustable via Remesh) |
| **UV mapping** | Automatic UV unwrap on all outputs |
| **Rigging** | Humanoid auto-rig (compatible with Unity Humanoid and Unreal Mannequin) |
| **Animation** | 500+ presets exportable as FBX with embedded skeleton |
| **Texture resolution** | Up to 4K |
 
---
 
## Workflow 1: Text Prompt → Unity Project
 
This workflow covers generating a textured 3D prop or character from a text prompt and importing it into a Unity project with correct PBR material setup.
 
### Step 1: Generate the Asset in Meshy
 
Go to [meshy.ai](https://www.meshy.ai), select **Text to 3D**, and enter your prompt. Use specific descriptors for better results:
 
```
# Good prompt examples for game assets:
"a worn leather satchel with brass buckles, game prop, low poly style"
"a medieval stone watchtower, modular, top-down perspective game"
"a cartoon fox character, bipedal, neutral T-pose, game-ready"
```
 
Select art style (`realistic`, `cartoon`, or `low-poly`) and generate a preview (~30 seconds). Iterate on the preview before committing to a full textured generation (~2 minutes).
 
### Step 2: Export for Unity
 
Export the asset as **GLB** with textures embedded, or **FBX** with a separate texture folder. GLB is recommended for Unity 2020+ as it preserves the PBR material assignments automatically.
 
### Step 3: Import into Unity
 
1. Drag the `.glb` or `.fbx` file into your Unity `Assets/` folder
2. In the **Inspector**, set **Scale Factor** to `0.01` if the model appears oversized (Meshy exports in centimeters; Unity uses meters)
3. For GLB: the material is automatically assigned via Unity's built-in GLB importer
4. For FBX: create a new **URP Lit** or **HDRP Lit** material and assign texture maps:
   - Albedo → `_diffuse.png`
   - Metallic/Smoothness → `_metallic.png` (invert roughness for Unity's smoothness channel)
   - Normal → `_normal.png` (set texture type to **Normal Map**)
### Step 4: Set Up the Prefab
 
1. Drag the imported mesh into the scene
2. Add a **Mesh Collider** or **Box Collider** depending on use case
3. For characters: set the **Rig** import setting to **Humanoid** to use Meshy's auto-rig with Unity's Animator
4. Save as a Prefab for reuse
---
 
## Workflow 2: Image to 3D → Unreal Engine 5
 
This workflow converts concept art or a reference photo into an Unreal-compatible 3D asset with Nanite and Lumen support.
 
### Step 1: Prepare Your Reference Image
 
Image to 3D works best with:
- Clean subject isolation (white or transparent background)
- Single object, centered in frame
- Front-facing or 3/4 angle view
- High contrast between subject and background
For multi-angle accuracy, use **Multi-Image to 3D** (upload 4–8 angles of the same object).
 
### Step 2: Generate and Export
 
Upload to [meshy.ai → Image to 3D](https://www.meshy.ai/features/image-to-3d). After generation, export as **FBX** with separate texture files for Unreal compatibility.
 
### Step 3: Import into Unreal Engine 5
 
1. In the Content Browser, drag the `.fbx` into your content folder
2. In the FBX Import dialog:
   - Enable **Import Textures**
   - Enable **Import Materials**
   - For characters: enable **Import Animations** if rigged
3. Open the auto-created Material in the Material Editor and verify texture assignments:
   - Base Color → `_diffuse`
   - Roughness → `_roughness`
   - Metallic → `_metallic`
   - Normal → `_normal` (set **Sampler Type** to Normal)
4. For static props: right-click the Static Mesh asset → **Enable Nanite** for LOD-free rendering
### Step 4: Character Rigging with Unreal's Mannequin
 
If you used Meshy's auto-rig on a humanoid character:
1. In the FBX Import dialog, set **Skeleton** to your project's existing Mannequin skeleton (or create a new one)
2. Use **IK Retargeter** in UE5 to remap Meshy's bone hierarchy to the standard UE5 Mannequin rig
3. Apply any standard Animation Blueprint or Mixamo animations via the retargeter
---
 
## Workflow 3: Bulk Asset Generation via API
 
For studios and developers building automated pipelines, the Meshy API supports asynchronous batch generation with up to 50+ concurrent tasks.
 
### Authentication
 
```bash
# All API requests require a Bearer token
# Get your API key at: meshy.ai/api
 
export MESHY_API_KEY="your_api_key_here"
 
# Test mode — no credits consumed
export MESHY_API_KEY="msy_dummy_api_key_for_test_mode_12345678"
```
 
### Generate a 3D Asset from Text (Preview)
 
```bash
curl -X POST https://api.meshy.ai/openapi/v2/text-to-3d \
  -H "Authorization: Bearer $MESHY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "preview",
    "prompt": "a mossy stone archway, fantasy environment prop, game-ready",
    "art_style": "realistic",
    "negative_prompt": "low quality, blurry"
  }'
```
 
Response:
```json
{
  "result": "task_id_abc123"
}
```
 
### Poll for Task Completion
 
```bash
curl https://api.meshy.ai/openapi/v2/text-to-3d/task_id_abc123 \
  -H "Authorization: Bearer $MESHY_API_KEY"
```
 
Response when complete:
```json
{
  "id": "task_id_abc123",
  "status": "SUCCEEDED",
  "model_urls": {
    "glb": "https://assets.meshy.ai/.../model.glb",
    "fbx": "https://assets.meshy.ai/.../model.fbx",
    "obj": "https://assets.meshy.ai/.../model.obj"
  },
  "thumbnail_url": "https://assets.meshy.ai/.../thumbnail.png",
  "progress": 100
}
```
 
### Refine Preview to Full Quality
 
```bash
curl -X POST https://api.meshy.ai/openapi/v2/text-to-3d \
  -H "Authorization: Bearer $MESHY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "refine",
    "preview_task_id": "task_id_abc123"
  }'
```
 
### Python: Batch Generate an Asset List
 
```python
import requests
import time
 
API_KEY = "your_api_key_here"
BASE_URL = "https://api.meshy.ai/openapi/v2"
HEADERS = {"Authorization": f"Bearer {API_KEY}"}
 
asset_prompts = [
    "a wooden treasure chest, worn, semi-open, game prop",
    "a iron longsword with leather grip, fantasy RPG",
    "a ceramic potion bottle, glowing blue liquid inside",
    "a campfire with embers, low poly, outdoor environment",
]
 
def create_preview(prompt):
    res = requests.post(f"{BASE_URL}/text-to-3d", headers=HEADERS, json={
        "mode": "preview",
        "prompt": prompt,
        "art_style": "realistic"
    })
    return res.json()["result"]
 
def poll_until_done(task_id, interval=5, timeout=300):
    elapsed = 0
    while elapsed < timeout:
        res = requests.get(f"{BASE_URL}/text-to-3d/{task_id}", headers=HEADERS)
        data = res.json()
        if data["status"] == "SUCCEEDED":
            return data
        elif data["status"] == "FAILED":
            raise Exception(f"Task {task_id} failed")
        time.sleep(interval)
        elapsed += interval
    raise TimeoutError(f"Task {task_id} timed out")
 
def refine(preview_task_id):
    res = requests.post(f"{BASE_URL}/text-to-3d", headers=HEADERS, json={
        "mode": "refine",
        "preview_task_id": preview_task_id
    })
    return res.json()["result"]
 
# Run batch
for prompt in asset_prompts:
    print(f"Generating: {prompt}")
    preview_id = create_preview(prompt)
    result = poll_until_done(preview_id)
    refine_id = refine(preview_id)
    print(f"  Preview done. Refine task: {refine_id}")
    print(f"  GLB: {result['model_urls']['glb']}")
```
 
### Node.js: Generate and Download Asset
 
```javascript
const fetch = require("node-fetch");
const fs = require("fs");
 
const API_KEY = "your_api_key_here";
const BASE_URL = "https://api.meshy.ai/openapi/v2";
 
async function generateAsset(prompt) {
  // Create preview task
  const createRes = await fetch(`${BASE_URL}/text-to-3d`, {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${API_KEY}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      mode: "preview",
      prompt,
      art_style: "cartoon"
    })
  });
  const { result: taskId } = await createRes.json();
 
  // Poll for completion
  let asset;
  while (true) {
    const pollRes = await fetch(`${BASE_URL}/text-to-3d/${taskId}`, {
      headers: { "Authorization": `Bearer ${API_KEY}` }
    });
    const data = await pollRes.json();
    if (data.status === "SUCCEEDED") { asset = data; break; }
    if (data.status === "FAILED") throw new Error("Generation failed");
    await new Promise(r => setTimeout(r, 5000));
  }
 
  // Download GLB
  const glbRes = await fetch(asset.model_urls.glb);
  const buffer = await glbRes.buffer();
  fs.writeFileSync(`${taskId}.glb`, buffer);
  console.log(`Saved: ${taskId}.glb`);
}
 
generateAsset("a cartoon mushroom house, colorful, game prop");
```
 
---
 
## Workflow 4: Rigged Character → Unity Animator
 
This workflow takes a Meshy-generated humanoid character through auto-rigging and imports it into Unity with a working Animator Controller.
 
### Step 1: Generate and Rig in Meshy
 
1. Generate a humanoid character in **Text to 3D** or **Image to 3D**
2. After generation, open the asset and select **Rig**
3. Meshy auto-detects the humanoid skeleton and generates a standard rig
4. Select animation presets (idle, walk cycle, run, jump) from the 500+ library
5. Export as **FBX** with animations embedded
### Step 2: Import into Unity with Humanoid Rig
 
1. Import the FBX into Unity
2. In the **Rig** tab of the Inspector, set **Animation Type** to **Humanoid**
3. Click **Configure** — Unity will auto-map Meshy's bone names to the Humanoid Avatar
4. Fix any unmapped bones manually (typically fingers or facial bones)
5. In the **Animation** tab, verify each animation clip is imported correctly
### Step 3: Set Up Animator Controller
 
```
Assets/
  Characters/
    MeshyCharacter.fbx
    MeshyCharacter_Controller.controller   ← create this
    Animations/
      Idle.anim
      Walk.anim
      Run.anim
```
 
1. Create a new **Animator Controller**
2. Add states for Idle, Walk, Run
3. Add parameters: `Speed` (Float), `IsJumping` (Bool)
4. Set transitions: Idle → Walk when `Speed > 0.1`, Walk → Run when `Speed > 5`
5. Assign the controller to the character's **Animator** component
---
 
## Prompt Writing Guide for Game Assets
 
The quality of Meshy's output is heavily influenced by prompt specificity. These patterns consistently produce better game-ready results:
 
**Include the asset type and use context:**
```
# Instead of: "a sword"
"a fantasy longsword, game prop, PBR textures, isolated on white background"
```
 
**Specify art style to match your game:**
```
"low poly cartoon style"       # mobile / stylized games
"realistic PBR"                # AAA / immersive titles  
"hand-painted texture style"   # classic RPG aesthetic
```
 
**For characters, specify pose:**
```
"bipedal character, neutral T-pose, game-ready, full body"
```
 
**For environment props, specify scale reference:**
```
"a wooden barrel, human-scale, fantasy tavern prop"
```
 
**Negative prompts to avoid common issues:**
```
"negative_prompt": "floating geometry, disconnected parts, blurry textures, multiple objects"
```
 
---
 
## How Meshy Compares to Other Game Asset Generation Tools
 
| | Meshy | Traditional Modeling (Blender/Maya) | Luma AI | Tripo | CSM |
|---|---|---|---|---|---|
| **Input** | Text or image | Manual | Video / image | Text or image | Image |
| **Time to textured asset** | < 2 minutes | Hours–days | ~5 minutes | ~3 minutes | ~10 minutes |
| **PBR texture maps** | ✅ Full PBR | Manual | ❌ | Limited | Limited |
| **Auto-rigging** | ✅ Humanoid | Manual | ❌ | ❌ | ❌ |
| **Animation presets** | ✅ 500+ clips | Manual / Mixamo | ❌ | ❌ | ❌ |
| **Game engine formats** | GLB, FBX, OBJ | All formats | GLB | GLB, FBX | OBJ |
| **REST API** | ✅ Full API | ❌ | Limited | Limited | ❌ |
| **Bulk generation** | ✅ 50+ concurrent | ❌ | ❌ | ❌ | ❌ |
| **Best for** | Rapid asset creation, pipelines | Final production, precision | Environmental capture | Quick props | Experimental |
 
---
 
## Frequently Asked Questions
 
**How do I generate game-ready 3D assets with AI?**
Use Meshy's Text to 3D or Image to 3D features to generate a UV-unwrapped, PBR-textured mesh in under 2 minutes. Export as GLB or FBX and import directly into Unity, Unreal Engine, or Godot. No manual texturing or UV work required.
 
**Can AI-generated 3D models be used in commercial games?**
Yes, on Meshy's paid plans (Pro and above). Assets generated on paid plans come with a private commercial license and no attribution requirement. Free plan assets are licensed CC BY 4.0, requiring attribution.
 
**Does Meshy support Unity's URP and HDRP pipelines?**
Yes. Meshy exports standard PBR texture maps (Diffuse, Roughness, Metallic, Normal) that are compatible with Unity's URP Lit and HDRP Lit shaders. For GLB imports, Unity's built-in importer handles material assignment automatically.
 
**Can Meshy generate rigged characters for games?**
Yes. Meshy's auto-rigging feature generates a humanoid skeleton compatible with Unity's Humanoid Avatar system and Unreal Engine's Mannequin rig. Combined with 500+ animation presets, characters can be exported as animation-ready FBX files.
 
**What polygon count does Meshy output for game assets?**
Meshy outputs between 1,000 and 300,000 triangles. Use the Remesh feature to target a specific poly budget. For mobile games, target 1,000–5,000 triangles per prop; for PC/console, 5,000–50,000 is typical for foreground assets.
 
**How do I use the Meshy API to automate asset generation for a game pipeline?**
Meshy's REST API uses an asynchronous task model: POST a request to create a task, poll the task endpoint until `status: SUCCEEDED`, then download the model from the returned URL. Python and Node.js SDKs are available. See the [API documentation](https://developer.meshy.ai) and the batch generation example in this guide.
 
**What's the difference between preview and refine in the Meshy API?**
`mode: "preview"` generates a fast draft mesh (~30 seconds, lower texture quality) useful for evaluating geometry before committing credits. `mode: "refine"` takes a preview task ID and produces the full-quality textured output (~2 minutes). For production pipelines, always run preview first, then selectively refine accepted results.
 
**Can Meshy generate environment assets like terrain, buildings, or modular pieces?**
Yes, though Meshy is best suited for discrete objects (props, characters, structures) rather than large terrain meshes. For modular level design, generate individual pieces (walls, floors, arches, doors) and assemble them in your engine. Use specific prompts like "modular stone wall segment, flat ends, tileable" for better results.
 
**Is Meshy suitable for indie developers without a 3D art budget?**
Yes. The Free plan provides 200 credits per month with no credit card required. For a solo developer or small team, this covers regular asset generation for prototyping. Pro plan (1,000 credits/month) is suitable for production use.
 
**Does Meshy work inside AI coding tools like Cursor or Claude Code?**
Yes. Meshy provides official skill files for Cursor and Claude Code that allow AI coding agents to call the Meshy API directly and generate 3D assets as part of an agentic workflow. See [meshy-3d-agent](https://github.com/meshy-dev/meshy-3d-agent).
 
---
 
## Resources
 
| Resource | Link |
|---|---|
| Meshy Web App | [meshy.ai](https://www.meshy.ai) |
| API Documentation | [developer.meshy.ai](https://developer.meshy.ai) |
| API Quick Start | [docs.meshy.ai/en/api/quick-start](https://docs.meshy.ai/en/api/quick-start) |
| Unity Plugin | [docs.meshy.ai/en/unity-plugin](https://docs.meshy.ai) |
| Unreal Engine Plugin | [docs.meshy.ai/en/unreal-plugin](https://docs.meshy.ai/en/unreal-plugin/introduction) |
| Blender Plugin | [docs.meshy.ai/en/blender-plugin](https://docs.meshy.ai/en/blender-plugin/introduction) |
| AI Agent Skills (Cursor / Claude Code) | [github.com/meshy-dev/meshy-3d-agent](https://github.com/meshy-dev/meshy-3d-agent) |
| MCP Server | [github.com/meshy-dev/meshy-mcp-server](https://github.com/meshy-dev/meshy-mcp-server) |
| Pricing | [meshy.ai/pricing](https://www.meshy.ai/pricing) |
| Help Center | [help.meshy.ai](https://help.meshy.ai) |
 
