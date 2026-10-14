---
name: threed
description: "3D model creation for AI agents, done as an engineering discipline: procedural, Blender-scripted, and AI-generated geometry that actually validates. TRACK 1 - CRAFT (the 70%): the mesh fundamentals agents keep getting wrong - manifold/watertight geometry, normal orientation, real-world scale and units, topology (quads vs tris, when each), polycount budgets per destination, origin/pivot discipline, and a 3D-slop death list (non-manifold exports, inverted normals, unitless models, detail before silhouette, exports never reimported). TRACK 2 - TOOLBOX: every workable path to a model - parametric CAD (OpenSCAD CSG, CadQuery/build123d), Blender headless bpy (numbered build scripts, review renders, bake HIGH-to-LOW, export manifests, roundtrip reimport), three.js-ready web assets, AI text/image-to-3D generation APIs (concept-image-first, price-before-spend, staged game-ready chain, drift checks between steps), programmatic repair and optimization (trimesh, manifold3d, gltf-transform, pymeshlab), and 3D-print readiness (wall thickness, overhangs, orientation, tolerances, slicer handoff). Hard rules throughout: validate by numbers, verify by renders you actually look at, silhouette before detail, know the size before you generate, every export gets a roundtrip. Use for ANY 3D model task: 'make me a 3D model', 'model this in Blender', 'generate a 3D asset', 'convert / repair / optimize this mesh', 'export for Unity / web / printing', 'is this mesh printable', 'decimate to budget', 'turn this drawing into a model' - even when the user just says 'make it 3D'."
version: 1.0.0
license: MIT for the synthesis; upstream content keeps its own licenses - see ATTRIBUTION.md
---

# threed — 3D models as verified geometry, not hopeful files

`threed` builds models the way an engineer ships parts and a technical artist ships assets: the mesh is **data** — manifold, oriented, scaled, budgeted — the agent's blindness is compensated by two verification channels (numbers from mesh gates, renders the agent actually looks at), and nothing is called done that hasn't survived a roundtrip.

## The threed contract

1. **Purpose decides everything.** Engine asset, web scene, 3D print, icon, AR — the destination sets the budget (10–50k tris game-ready, <100k web hero, sane-but-unlimited print), the format (GLB / FBX / STL / 3MF / USDZ), and the gates. No purpose stated → ask before modeling.
2. **Know the size before you build or generate.** Scale is the #1 failure of AI-generated 3D. Real-world units declared up front ("80 mm tall", "1.8 m character"), bounding box verified against intent before anything else.
3. **Silhouette before detail.** Most wasted work is detail added on top of wrong proportions. Blockout → silhouette check (rendered, compared) → only then detail. A beautiful surface on a wrong shape is scrap.
4. **The mesh is data: manifold, oriented, scaled, budgeted.** Non-manifold edges, inverted normals, zero-area faces, duplicate vertices, unitless sizes — each is a gate the model passes with numbers, not vibes.
5. **Two-channel verification, always.** Numbers (mesh report: watertight, winding, volume, bbox, face count) AND renders from fixed views that the agent actually inspects. Script success ≠ model success; "I wrote the file" is not "the model is right".
6. **Every export gets a roundtrip.** Reimport the GLB/FBX/STL into a blank scene, probe what actually arrived (counts, UVs, materials, scale), render one check. Unverified exports don't ship.

## Routing table

Read only what the task needs:

| When | Read |
| --- | --- |
| Choosing topology, budgets, units, validation gates — or cutting 3D slop | `references/craft.md` |
| Modeling in/through Blender: headless bpy, numbered build scripts, review renders, baking, export | `references/blender.md` |
| Exact-dimension, parametric, or printable parts: OpenSCAD, CadQuery, build123d | `references/cad.md` |
| Formats, conversion, optimization, programmatic repair: OBJ/STL/GLB/FBX/3MF/USDZ, trimesh, gltf-transform | `references/formats.md` |
| AI-generated 3D: text/image-to-3D APIs, sourcing ladder, pricing discipline, staged game-ready chain | `references/generative.md` |
| 3D printing: printability gates, orientation, wall thickness, tolerances, slicer handoff | `references/printing.md` |
| Ready-to-adapt builds (R01–R14, dual-stack) | `references/recipes.md` |

## The one-glance path matrix

| Task | First call |
| --- | --- |
| Precise/mechanical part (bracket, enclosure, spacer, jig) | Parametric CAD (`cad.md`) |
| Organic or complex asset from a reference image | Blender headless, gated phases (`blender.md`) |
| Textured asset from a description or picture, fast | Generation API, concept-first (`generative.md`), then repair + decimate |
| Convert / optimize / validate an existing mesh | `formats.md` (trimesh + gltf-transform) |
| Prepare a model for 3D printing | `printing.md` |
| Web hero object / three.js asset | `formats.md` — web budget, draco/meshopt, KTX2 |
| Turn 2D drawings / turnaround into a model | `blender.md` reference-plane workflow |
| Multi-view images of one subject (orbit, turnaround) | `generative.md` — 3D-intermediary route, never per-angle guessing |

Rule of thumb: exact dimensions and holes → CAD code. Organic shapes from references → Blender scripts. Textured near-ready assets fast → generation API, then repair and decimate (it is a draft, never a final). Never mix: don't sculpt a bracket, don't CAD a dragon.

## The loop in one breath

**Intent** (destination + budget + size + format) → **Source** (sourcing ladder: existing → generate → Blender → pure code) → **Build** (parametric or scripted — reproducible scripts, not hand edits) → **Validate** (mesh numbers against gates) → **Review** (fixed-view renders the agent inspects; user shown before paid detail) → **Export** (destination format + manifest) → **Roundtrip** (reimport, probe, render, report real numbers).

## Hard rules

- Never claim a model from code success or file existence — numbers + renders, or it didn't happen.
- AI-generated meshes are **drafts**: analyze + repair (manifold, normals, degenerate) + decimate to budget before any delivery.
- Price before spend: paid generation gets a quote/dry-run first, and an approval gate before each costly stage (the concept is cents; the full chain is dollars).
- Quads exist only in FBX and DCC formats; glTF, OBJ, STL are triangles — never promise quad topology in a GLB.
- Scale, pivot, and normals drift between pipeline steps (generation → unwrap → retexture → rig): re-verify after every stage.
- A print is started by the **user**, not the agent — analysis, orientation, and preview are the agent's job; the button is theirs.
- One variable per rerun; seeds fixed for reproducibility; regenerate the failing view or part, not the whole asset.
- `.blend` is not an interchange format — deliver exports, keep the project file as source.

## Out of scope

- Animating and interacting with 3D scenes on the web (the `motion` skill's three.js track).
- Editing/rendering video produced from 3D (the `video` skill).
- Terrain surveying, GIS point clouds, photogrammetry capture pipelines.
- Deep rigging/animation craft — covered here only as delivery contracts and export gates.

## Sources

This skill is an original synthesis (MIT) merging upstream skills — `achimala/dream-loop` (MIT), `scenario-labs/skills` (MIT), `majidmanzarpour/blender-game-skills` (MIT), `heyixuan2/bambu-studio-ai` (MIT), `arjun988/blender-skills` (MIT), `viettranx/3dviz-pro-max` (MIT), `samyost1/3dicon` (MIT), `icesixgod/awesome-astra-blender-characters` (MIT), `RobLe3/cc-blender-skill` (MIT), plus concept-only credit to `flowful-ai/cad-skill` (PolyForm NC — concepts acknowledged, content not merged). Package versions live-verified 2026-10-15. Full credits and licenses: `ATTRIBUTION.md`.
