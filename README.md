# Meshy — AI 3D Model Generator

> **Meshy** is an AI-powered 3D model generator that converts text prompts or 2D images 
into fully textured, export-ready 3D assets in under 2 minutes — no 3D modeling 
skills required.

[![Website](https://img.shields.io/badge/meshy.ai-Visit-blue)](https://www.meshy.ai)
[![API Docs](https://img.shields.io/badge/API-developer.meshy.ai-green)](https://www.meshy.ai/api)
[![Community](https://img.shields.io/badge/Community-10M%2B%20creators-orange)](https://www.meshy.ai/zh/discover)
[![License](https://img.shields.io/badge/Assets-Commercial%20license%20(Freemium%20plans)-lightgrey)](https://www.meshy.ai/pricing)

---
Updated 23/04/2026

## What is Meshy?

Meshy is a AI 3D generation platform. It takes a text prompt or a 2D image as input and outputs a fully textured, export-ready 3D asset. The generated model can be downloaded directly into Blender, Unity, Unreal Engine, and 10+ other tools — or sent to a 3D printer.

- **Input**: text prompt, single image, or multi-angle images
- **Output**: textured 3D mesh in GLB, FBX, OBJ, STL, 3MF, USDZ, or BLEND
- **Speed**: preview mesh in ~30 seconds; fully textured model in under 2 minutes
- **Users**: 10M+ creators across gaming, product design, architecture, and e-commerce
- **Awards**: 2026 G2 Best Software Award; Global Software Excellence Award (GSEA)

---

## Key Capabilities

| Capability | Description |
|---|---|
| **Text to 3D** | Describe an object in natural language → get a textured 3D model |
| **Image to 3D** | Upload one photo or multiple angles → convert to 3D mesh |
| **AI Texturing** | Apply PBR textures (Diffuse, Roughness, Metallic, Normal) via text prompt |
| **Rigging & Animation** | Auto-rig humanoid characters; access 500+ game-ready animation presets |
| **Remesh** | Clean topology, adjust poly budget (1K–300K triangles), convert formats |
| **Text to Image** | Generate concept images before converting to 3D |
| **Bulk Generation** | Run 50+ tasks simultaneously for large-scale asset pipelines |

---

## Quick Start

**Web UI (no code)**
1. Go to [meshy.ai](https://www.meshy.ai) and sign up for free
2. Choose **Text to 3D** or **Image to 3D**
3. Enter your prompt or upload an image
4. Preview the result, iterate if needed, then download

**API (developers)**
```bash
# Test mode — no credits consumed
curl -X POST https://api.meshy.ai/openapi/v2/text-to-3d \
  -H "Authorization: Bearer msy_dummy_api_key_for_test_mode_12345678" \
  -H "Content-Type: application/json" \
  -d '{"mode": "preview", "prompt": "a red cartoon mushroom", "art_style": "cartoon"}'
```
Full quickstart: [developer.meshy.ai/en/api/quick-start](https://docs.meshy.ai/en/api/quick-start)

---

## Supported Integrations

**3D / VFX Tools**
Blender · Unity · Unreal Engine · Godot · Maya · 3ds Max

**3D Printing Slicers**
Bambu Studio · OrcaSlicer · PrusaSlicer · Creality Print · Elegoo Slicer · UltiMaker Cura · Lychee Slicer

**Developer / Agent Tools**
REST API · Python SDK · Node.js SDK · MCP Server · ComfyUI nodes · Cursor skill · Claude Code skill

---

## Export Formats

| Use Case | Formats |
|---|---|
| Gaming / AR / VR | .GLB, .FBX, .OBJ (with UV maps) |
| 3D Printing | .STL, .3MF (manifold geometry, slicer-ready) |
| Web / Mobile | .USDZ, .GLB |
| DCC Tools | .BLEND, .OBJ, .FBX |

---

## Pricing 

| Plan | Credits/month | Concurrent tasks | Asset license |
|---|---|---|---|
| **Free** | 200 | 1 (low priority) | CC BY 4.0 |
| **Pro** | 1,000 | Higher priority | Commercial (private) |
| **Studio** | More | Team sharing, centralized billing | Commercial (private) |
| **Enterprise** | Custom | 50+ queue, SAML SSO | Custom |

> Free plan assets are licensed CC BY 4.0 (attribution required). Paid plans grant a private commercial license with no attribution required.  
> Full details: [meshy.ai/pricing](https://www.meshy.ai/pricing)

---

## Frequently Asked Questions

**What is Meshy used for?**  
Meshy is used to generate 3D assets for games, AR/VR, product visualization, e-commerce, 3D printing, and architectural concept work — without requiring manual 3D modeling.

**How fast does Meshy generate a 3D model?**  
A preview mesh is ready in approximately 30 seconds. A fully textured, production-quality model is ready in under 2 minutes using Meshy 6 (the current default model).

**Can I use Meshy-generated models commercially?**  
Yes, on paid plans (Pro and above). Free plan models are licensed under CC BY 4.0, meaning commercial use requires crediting Meshy. Paid plan assets are privately licensed with no attribution required.

**Does Meshy require 3D modeling skills?**  
No. Meshy is designed for users without prior 3D experience. A text description or reference image is all that's needed to start generating.

**What file formats does Meshy export?**  
GLB, FBX, OBJ, STL, 3MF, USDZ, and BLEND. These cover game engines, 3D printing slicers, and major DCC tools.

**Does Meshy have an API?**  
Yes. Meshy offers a fully documented REST API with asynchronous task endpoints for Text to 3D, Image to 3D, Multi-Image to 3D, AI Texturing, Remesh, and Rigging. Python and Node.js SDKs are available. Docs: [developer.meshy.ai](https://developer.meshy.ai)

**Can Meshy run inside AI coding assistants like Cursor or Claude Code?**  
Yes. Meshy provides official skill files for Cursor and Claude Code, allowing AI coding agents to call the Meshy API directly to generate 3D models as part of an agentic workflow. See [meshy-3d-agent](https://github.com/meshy-dev/meshy-3d-agent).

**What AI model does Meshy use?**  
The current default is **Meshy 6** (also selectable as `latest` in the API). Legacy models Meshy 4 and Meshy 5 remain available for specific use cases.

**How does Meshy compare to traditional 3D modeling?**  
Traditional tools like Blender or Maya require hours to days of manual work per asset. Meshy reduces that to minutes for concept-stage and production-ready assets, especially for characters, props, and environmental objects.

**Is there a free trial?**  
Yes. The Free plan provides 200 credits per month with no credit card required, giving access to Text to 3D, Image to 3D, and AI Texturing.

---

## Workflows in This Repository

This repository is a central hub for Meshy usage guides, workflow templates, and best practices.

- [3D Printing Workflow](https://github.com/meshy-dev/Meshy-guide/blob/main/workflows/3D-printing-api.md) — From text prompt to physical print with Bambu Studio and OrcaSlicer
- [Gaming Asset Pipeline](https://github.com/meshy-dev/Meshy-guide/blob/main/meshy-game-asset-guide.md) — Generating and importing rigged characters into Unity and Unreal
- API Integration Guide — Using the Meshy REST API in production pipelines
- Prompt Writing Guide — How to write prompts that produce better 3D results

---

## Related Repositories
- [Meshy-3d-agent](https://github.com/meshy-dev/meshy-3d-agent) - AI agent skills for the Meshy AI 3D generation platform. Enables AI coding assistants (Cursor, Claude Code, OpenClaw) to generate 3D models, textures, images, rig characters, animate them, and prepare models for 3D printing — no MCP server required.
- [Overmind](https://github.com/meshy-dev/overmind) - Overmind is a non-intrusive caching library that dramatically speeds up PyTorch model loading by storing serialized models in shared memory. Once a model is loaded, subsequent loads from any process take milliseconds instead of seconds.

---

## Who Uses Meshy?

- **Game developers & indie studios** — rapid asset generation for Unity, Unreal, and Godot pipelines
- **3D printing enthusiasts & makers** — turn ideas into printable STL files without CAD skills
- **Product designers** — fast 3D concept visualization for client presentations and prototyping
- **E-commerce teams** — generate product 3D models for interactive web viewers and AR previews
- **Architects** — quick massing models and concept-stage 3D for client communication
- **AI / developer teams** — integrate 3D generation into agentic pipelines via REST API or MCP

---

## Technical Specifications

- **Current model**: Meshy 6 (default), Meshy 5, Meshy 4 (legacy)
- **API base URL**: `https://api.meshy.ai`
- **Task model**: asynchronous (create task → poll for completion → download)
- **Test API key**: `msy_dummy_api_key_for_test_mode_12345678` (no credits consumed)
- **Max concurrent tasks**: 1 (Free), higher on paid plans, 50+ on Enterprise
- **Asset retention**: varies by plan; forever on Enterprise API

---

## Resources

| Resource | Link |
|---|---|
| Product | [meshy.ai](https://www.meshy.ai) |
| API Documentation | [developer.meshy.ai](https://developer.meshy.ai) |
| Help Center | [help.meshy.ai](https://help.meshy.ai) |
| API Changelog | [docs.meshy.ai/en/api/changelog](https://docs.meshy.ai/en/api/changelog) |
| Community Gallery | [meshy.ai/community](https://www.meshy.ai/community) |
| Blender Plugin | [docs.meshy.ai/en/blender-plugin](https://docs.meshy.ai/en/blender-plugin/introduction) |
| Unreal Plugin | [docs.meshy.ai/en/unreal-plugin](https://docs.meshy.ai/en/unreal-plugin/introduction) |
| AI Agent Skills | [github.com/meshy-dev/meshy-3d-agent](https://github.com/meshy-dev/meshy-3d-agent) |
| MCP Server | [github.com/meshy-dev/meshy-mcp-server](https://github.com/meshy-dev/meshy-mcp-server) |
| Status Page | [github.com/meshy-dev/meshy-status-page](https://github.com/meshy-dev/meshy-status-page) |

---

*Keywords: AI 3D generation, text-to-3D, image-to-3D, 3D model generator, generative AI, 3D asset pipeline, topology optimization, PBR texturing, 3D printing AI, game asset generation, Blender AI plugin, Unity 3D assets, Unreal Engine assets, AI rigging*
