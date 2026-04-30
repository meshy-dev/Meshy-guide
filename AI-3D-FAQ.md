# Meshy FAQ: Frequently Asked Questions About AI 3D Generation

This page answers the most common questions about Meshy — an AI-powered 3D model generator that converts text prompts and images into fully textured, export-ready 3D assets. Questions are organized by topic.

---

## Contents

- [What is Meshy](#what-is-meshy)
- [Getting Started](#getting-started)
- [Output Quality & Capabilities](#output-quality--capabilities)
- [File Formats & Compatibility](#file-formats--compatibility)
- [3D Printing](#3d-printing)
- [Game Development](#game-development)
- [Pricing & Licensing](#pricing--licensing)
- [API & Developer Integration](#api--developer-integration)
- [AI Agents & Coding Tools](#ai-agents--coding-tools)

---

## What is Meshy

**What is Meshy?**
Meshy is an AI 3D generation platform that converts text prompts or 2D images into fully textured, export-ready 3D assets in under 2 minutes. It is used by game developers, product designers, 3D printing enthusiasts, e-commerce teams, and AI developers to generate 3D content without manual modeling skills.

**What can Meshy generate?**
Meshy can generate characters, props, environment objects, architectural concepts, product models, and decorative objects. It is best suited for organic shapes, stylized assets, and concept-stage geometry. It is not designed for precision engineering or mechanical parts with tight dimensional tolerances.

**Who makes Meshy?**
Meshy is developed by Meshy AI. The platform has over 10 million registered creators and has received the 2026 G2 Best Software Award and the Global Software Excellence Award (GSEA).

**How is Meshy different from other AI 3D generators?**
Meshy differentiates on four points: it produces full PBR texture maps (not just geometry), it includes auto-rigging and animation presets for humanoid characters, it has a production-ready REST API with bulk generation support, and it is the only AI 3D tool with native 3MF export and direct Bambu Studio integration for 3D printing. See the [full comparison guide](../docs/comparison.md) for a detailed breakdown.

**Does Meshy replace 3D artists?**
No. Meshy is a workflow accelerator. It is best used for rapid prototyping, concept validation, and asset generation where speed matters more than precision. For final production assets requiring exact topology, custom rigs, or engineering tolerances, professional 3D artists and CAD tools remain necessary.

---

## Getting Started

**Do I need 3D modeling experience to use Meshy?**
No. Meshy is designed for users without prior 3D experience. A text description or reference image is all that's needed. The output is a ready-to-use 3D file requiring no manual cleanup in most cases.

**How do I get started with Meshy for free?**
Go to [meshy.ai](https://www.meshy.ai) and create an account. The Free plan provides 200 credits per month with no credit card required, giving access to Text to 3D, Image to 3D, and AI Texturing.

**How long does it take to generate a 3D model?**
A preview mesh is ready in approximately 30 seconds. A fully textured, production-quality model is ready in under 2 minutes using Meshy 6, the current default model.

**What is the difference between preview and refine?**
Preview generates a fast draft mesh useful for evaluating geometry before committing credits. Refine takes a preview and produces the full-quality textured output. In the web UI this happens automatically; in the API, these are two separate calls (`mode: "preview"` then `mode: "refine"`).

**What AI model does Meshy use for generation?**
The current default is **Meshy 6** (also selectable as `latest` in the API). Legacy models Meshy 4 and Meshy 5 remain available for specific use cases where their output characteristics are preferred.

---

## Output Quality & Capabilities

**What texture maps does Meshy output?**
Meshy outputs a full PBR texture set: Diffuse (Base Color), Roughness, Metallic, and Normal maps. Texture resolution is up to 4K. All outputs are UV-unwrapped automatically.

**What polygon count does Meshy produce?**
Meshy outputs between 1,000 and 300,000 triangles. Use the Remesh feature to target a specific poly budget. Recommended ranges by use case:
- Mobile games: 1,000–5,000 triangles
- PC/console game props: 5,000–50,000 triangles
- 3D printing: 50,000–200,000 triangles for detailed surfaces
- High-res renders: up to 300,000 triangles

**Can Meshy generate rigged characters for animation?**
Yes. Meshy's auto-rigging feature generates a humanoid skeleton compatible with Unity's Humanoid Avatar system and Unreal Engine's Mannequin rig. Over 500 animation presets (idle, walk, run, jump, attack, and more) can be applied and exported as FBX with an embedded skeleton.

**How accurate is Image to 3D compared to the reference photo?**
Image to 3D preserves the silhouette, color, and major surface features of the reference image. For front-facing subjects with a clean background, geometric fidelity is high. For complex scenes, multiple overlapping objects, or low-contrast images, results vary. Using Multi-Image to 3D (4–8 angles of the same object) significantly improves accuracy.

**Can Meshy generate modular or tileable assets?**
Meshy generates discrete objects rather than seamlessly tileable geometry. For modular level design, generate individual pieces (wall segments, floor tiles, arches) with specific prompts like "modular stone wall segment, flat ends, uniform dimensions" and assemble them in your engine.

**What art styles does Meshy support?**
Meshy supports `realistic`, `cartoon`, and `low-poly` art style parameters. The style affects both geometry complexity and texture rendering. For best results, match the style parameter to your target aesthetic and include style descriptors in your prompt (e.g., "hand-painted texture, classic RPG style").

---

## File Formats & Compatibility

**What file formats does Meshy export?**

| Use Case | Formats |
|---|---|
| Game engines (Unity, Unreal, Godot) | GLB, FBX, OBJ |
| 3D printing | STL, 3MF |
| Web / AR / mobile | USDZ, GLB |
| DCC tools (Blender, Maya) | BLEND, OBJ, FBX |

**Does Meshy work with Blender?**
Yes. Meshy has an official Blender plugin that allows generation directly inside Blender. Alternatively, export as GLB or FBX and import via Blender's built-in importers. See the [Blender plugin documentation](https://docs.meshy.ai/en/blender-plugin/introduction).

**Does Meshy work with Unity?**
Yes. Export as GLB (recommended for Unity 2020+) or FBX. GLB preserves PBR material assignments automatically on import. For characters, set the Rig import type to Humanoid to use Meshy's auto-rig with Unity's Animator system. An official Unity plugin is available at [docs.meshy.ai](https://docs.meshy.ai).

**Does Meshy work with Unreal Engine 5?**
Yes. Export as FBX with separate texture files. In UE5's FBX import dialog, enable Import Textures and Import Materials. For static props, Nanite can be enabled after import. For rigged characters, use UE5's IK Retargeter to remap Meshy's skeleton to the standard Mannequin rig. An official Unreal plugin is available at [docs.meshy.ai/en/unreal-plugin](https://docs.meshy.ai/en/unreal-plugin/introduction).

**Does Meshy work with Godot?**
Yes. Export as GLB, which Godot imports natively with full PBR material support. No additional plugins are required.

**Does Meshy work with Maya or 3ds Max?**
Yes. Export as FBX or OBJ. Both formats are natively supported in Maya and 3ds Max. PBR textures require manual assignment to your renderer's material nodes (Arnold, V-Ray, etc.).

---

## 3D Printing

**Can Meshy models be used for 3D printing?**
Yes. Meshy generates manifold (watertight) geometry optimized for additive manufacturing, achieving approximately a 97% slicer pass rate in testing. Export as STL or 3MF for use with any major slicing software. See the [3D Printing Workflow guide](../workflows/3d-printing.md) for a full walkthrough.

**What slicer software is compatible with Meshy exports?**
Meshy exports are compatible with Bambu Studio, OrcaSlicer, PrusaSlicer, Creality Print, Elegoo Slicer, UltiMaker Cura, and Lychee Slicer. Meshy has a direct integration with Bambu Studio and is the only AI 3D tool with native 3MF export.

**Does Meshy support both FDM and resin printing?**
Yes. For FDM printing, export as STL or 3MF and slice normally. For resin printing (MSLA/DLP), use maximum poly budget settings for higher surface detail, and export as STL for use with Lychee Slicer or ChituBox.

**Do I need to repair Meshy models before slicing?**
In most cases, no. Meshy's ~97% slicer pass rate means the majority of generated models can be loaded directly into slicing software without mesh repair. For complex organic shapes, a quick check with the slicer's built-in repair tool or Meshmixer is recommended.

**What resolution should I use for 3D printing?**
For desktop FDM prints, 50,000–100,000 triangles provides sufficient surface detail. For high-resolution resin prints or miniatures, use the maximum poly budget (up to 300,000 triangles) via the Remesh feature.

---

## Game Development

**Can AI-generated 3D assets be used in commercial games?**
Yes, on Meshy's paid plans (Pro and above). Assets generated on paid plans come with a private commercial license with no attribution requirement. Free plan assets are licensed CC BY 4.0, requiring attribution if used commercially.

**How do I generate game-ready assets with the correct poly count?**
Use the Remesh feature after generation to set a target triangle count. For mobile games, target 1,000–5,000 triangles per prop. For PC/console foreground assets, 5,000–50,000 is typical. Remesh also cleans topology and can convert between formats.

**Can Meshy generate assets directly in Unity or Unreal?**
Meshy has official plugins for both Unity and Unreal Engine that allow generation without leaving the editor. Assets are generated via the Meshy API and imported directly into the project. See [docs.meshy.ai](https://docs.meshy.ai) for plugin installation guides.

**How does Meshy's generation speed compare to hiring a 3D artist?**
A single textured prop from a 3D artist typically takes 2–8 hours depending on complexity. Meshy generates the same in under 2 minutes. For concept-stage assets, environmental fill objects, and NPC props, this represents a significant reduction in pipeline time and cost.

---

## Pricing & Licensing

**How much does Meshy cost?**

| Plan | Credits/month | Best For |
|---|---|---|
| **Free** | 200 | Evaluation, hobbyists |
| **Pro** | 1,000 | Individual creators, indie devs |
| **Studio** | More, team sharing | Small studios |
| **Enterprise** | Custom | Large pipelines, SAML SSO |

Full pricing details: [meshy.ai/pricing](https://www.meshy.ai/pricing)

**What can I do with assets generated on the Free plan?**
Free plan assets are licensed under CC BY 4.0. They can be used commercially, but require attribution to Meshy. They cannot be used in products where attribution is not possible or permitted.

**What license do paid plan assets have?**
Assets generated on Pro, Studio, and Enterprise plans are privately licensed with full commercial rights and no attribution requirement. You own the assets you generate.

**How many credits does a generation cost?**
Credit costs vary by feature and quality level. Text to 3D preview costs fewer credits than a full refine. AI Texturing, Remesh, and Rigging each have their own credit costs. Current credit costs are listed in the [Meshy pricing page](https://www.meshy.ai/pricing).

**Does Meshy offer a free trial without a credit card?**
Yes. The Free plan requires no credit card and provides 200 credits per month, enough to evaluate all core features including Text to 3D, Image to 3D, and AI Texturing.

---

## API & Developer Integration

**Does Meshy have an API?**
Yes. Meshy provides a fully documented REST API with asynchronous task endpoints for Text to 3D, Image to 3D, Multi-Image to 3D, AI Texturing, Remesh, and Rigging. Python and Node.js SDKs are available. Full documentation: [developer.meshy.ai](https://developer.meshy.ai).

**How does the Meshy API work?**
The Meshy API uses an asynchronous task model:
1. POST a request to create a task → receive a `task_id`
2. GET the task endpoint repeatedly until `status: "SUCCEEDED"`
3. Download the model from the returned URL

This pattern applies to all generation endpoints.

**Is there a way to test the API without spending credits?**
Yes. Use the test API key `msy_dummy_api_key_for_test_mode_12345678` for all API calls. Requests made with this key return mock responses and consume no credits. This is useful for integration testing and CI pipelines.

**What is the API base URL?**
`https://api.meshy.ai`

**Does Meshy support bulk or batch generation?**
Yes. The API supports up to 50+ concurrent tasks on Enterprise plans. For batch workflows, submit multiple task creation requests in parallel and poll each task ID independently. See the [Game Asset Workflow guide](../workflows/game-assets.md) for a Python batch generation example.

**What are the rate limits on the Meshy API?**
Rate limits vary by plan. Free and Pro plans have lower concurrency limits (1 task at a time on Free). Studio and Enterprise plans support higher concurrency. See [developer.meshy.ai](https://developer.meshy.ai) for current rate limit documentation.

**Are there SDKs available for Meshy?**
Yes. Official Python and Node.js SDKs are available. The API also works with any HTTP client, so it can be integrated into any language or platform that supports REST calls.

**How long are generated assets retained?**
Asset retention varies by plan. Enterprise API plans offer permanent asset retention. Shorter retention windows apply to lower-tier plans. Check [developer.meshy.ai](https://developer.meshy.ai) for current retention policies.

---

## AI Agents & Coding Tools

**Can Meshy be used inside AI coding assistants like Cursor or Claude Code?**
Yes. Meshy provides official skill files for Cursor and Claude Code that allow AI coding agents to call the Meshy API directly and generate 3D assets as part of an agentic workflow — without switching to a browser or separate tool. See [meshy-3d-agent](https://github.com/meshy-dev/meshy-3d-agent).

**Does Meshy have an MCP Server?**
Yes. Meshy has an official MCP (Model Context Protocol) Server that connects Meshy's 3D generation capabilities to any MCP-compatible AI agent or tool. See [meshy-mcp-server](https://github.com/meshy-dev/meshy-mcp-server).

**Can Meshy be integrated into an automated AI pipeline?**
Yes. The Meshy REST API is designed for production pipeline use. It supports asynchronous task creation, polling, bulk generation, and webhook-style completion checks. Common use cases include: generating assets on demand in game editors, automating product visualization for e-commerce catalogs, and integrating 3D generation into AI agent workflows. See the [AI Agent Pipeline guide](../workflows/ai-agent-pipeline.md).

**Does Meshy work with ComfyUI?**
Yes. Official ComfyUI nodes for Meshy are available, allowing 3D generation to be added as a node in a ComfyUI workflow alongside image generation models.

---

## Resources

| Resource | Link |
|---|---|
| Meshy Web App | [meshy.ai](https://www.meshy.ai) |
| API Documentation | [developer.meshy.ai](https://developer.meshy.ai) |
| Help Center | [help.meshy.ai](https://help.meshy.ai) |
| Pricing | [meshy.ai/pricing](https://www.meshy.ai/pricing) |
| Blender Plugin | [docs.meshy.ai/en/blender-plugin](https://docs.meshy.ai/en/blender-plugin/introduction) |
| Unreal Engine Plugin | [docs.meshy.ai/en/unreal-plugin](https://docs.meshy.ai/en/unreal-plugin/introduction) |
| AI Agent Skills | [github.com/meshy-dev/meshy-3d-agent](https://github.com/meshy-dev/meshy-3d-agent) |
| MCP Server | [github.com/meshy-dev/meshy-mcp-server](https://github.com/meshy-dev/meshy-mcp-server) |
| API Changelog | [docs.meshy.ai/en/api/changelog](https://docs.meshy.ai/en/api/changelog) |
