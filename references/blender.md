# Blender as an agent tool — headless, scripted, verified

*Merged from: majidmanzarpour/blender-game-skills (numbered build scripts, gate protocol, review renders, bake/export/roundtrip), scenario-labs/skills (binary resolution, orbit-view renders, clay-render concept, DCC-fix lane), icesixgod/awesome-astra-blender-characters (version-chain discipline), RobLe3/cc-blender-skill (role routing, environment-check-first), arjun988/blender-skills (specialist-area map). Blender 5.2 LTS / 4.5 LTS current as of 2026-10-15.*

Blender is the universal DCC an agent can actually drive: scriptable end-to-end (bpy), free, import/export for every format, and it renders — which gives the agent eyes. The discipline: **everything through numbered scripts, nothing through hand edits** — a build that cannot be replayed is not a build, it's a memory.

## 1 · Runtime — resolve, verify, then trust

- **Resolve the binary once**, in this order: `$BLENDER_BIN` → `blender` on PATH → `/Applications/Blender.app/Contents/MacOS/Blender` (macOS never puts it on PATH). Verify: `blender --version` before the first job.
- **Headless is the default**: `blender --background --factory-startup --python script.py -- <args>`. `--factory-startup` gives a clean, reproducible scene state (no user prefs leaking in).
- **Version check first**: Blender 5.x changed several APIs (shader/principled inputs, some export ops, addon layouts). Probe `bpy.app.version` and adapt; don't assume 4.x docs hold on 5.x. LTS lines to expect in the wild: 5.2, 4.5, 4.2.
- **Renders without GPU**: Workbench engine renders fine headless on CPU/macOS/Windows; on a truly headless Linux box without GPU, pass Cycles with CPU threads. Review renders do not need beauty — they need truth.
- A live Blender via MCP (execute-Python tool) is optional and useful for inspection — but even then, build through the numbered scripts and save the same master file, so the build stays reproducible. In a live session: `exec(open(path).read())` and call functions, don't re-type logic.
- **Environment check before any work** (cc-blender pattern): binary found? `python3 -c "import numpy, PIL"` for the helpers? input files exist and are non-empty? Fail fast with a message naming the fix, not three steps in.

## 2 · The bpy mental model for agents

- Everything lives in `bpy.data` (meshes, materials, objects are data-blocks) and `bpy.context` (what's active). An object links to a mesh; many objects can share one mesh.
- **Scene graph ops**: `bpy.ops.mesh.primitive_cube_add()` (operator, context-sensitive) vs direct data API (`bpy.data.meshes.new()` + `bpy.data.objects.new()` + `collection.objects.link()`). Prefer the data API in scripts — operators depend on context and break headless.
- **Transforms**: `obj.location/rotation_euler/scale` are the object transform; `obj.matrix_world` is the composed result. Apply transforms (`obj.select_set(True); bpy.ops.object.transform_apply(...)`) only at export time — until then, transforms are cheap, editable data.
- **Origin**: `obj.data.validate()` sanity-checks a mesh; set origin deliberately (`bpy.ops.object.origin_set(type='ORIGIN_CURSOR')` or geometry math) — feet-on-origin for characters, base-on-origin for props.

## 3 · The numbered-build pattern

```
<asset>/
  build/00_init_master.py     # units, collections, ref planes, camera
  build/10_blockout.py        # primitives at real scale — silhouette lives here
  build/20_primary_forms.py   # joins, booleans, main bevels
  build/30_detail.py          # greebles, panels, small features
  build/40_materials.py       # PBR setup
  build/50_export.py          # export + manifest
  review/                     # renders and compare sheets
  master.blend
```

Rules that make it work:

- Each script **opens master.blend, deletes what it owns, rebuilds, saves**. Rerunning phase 30 never re-runs 10. Idempotent phases = cheap iteration.
- **Constants are meters/mm with a comment naming the reference view** they came from, or `# inferred`.
- Phase 10 must produce a **silhouette-checkable blockout** — render it before phase 20 exists. The gate between phases is rendered evidence compared against the reference *from the same camera and at gameplay pixel size*, never viewport glances. Most wasted work is detail added on wrong proportions; nothing gets detail until its silhouette passes.

## 4 · Review renders — the agent's eyes

One render script, several modes:

- **clay** — flat gray material, even light: reads silhouette, proportion, mass.
- **silhouette** — unlit white-on-black: pure shape comparison; overlay against the reference.
- **wire** — shows topology flow, density, pinches.
- **checker** — UV stretch and texel density.
- **turntable** — 16 cameras, 22.5° apart, one revolution: every face gets a frame; hidden junk surfaces here.
- **reference-matched camera** — for fidelity work, estimate the reference's azimuth/elevation/lens and render from THAT camera; compare side-by-side and as an overlay. Matched cameras make drift measurable ("the grip is 4% short"), not debatable.
- Compose a **contact sheet** (grid of the views) and look at it — one image, whole model. Regenerate only the failing view after a fix.
- Workbench for speed; Cycles only for final material/lighting checks. Gameplay pixel size (render at the size the asset appears on screen, e.g. 128 px tall) catches which details even exist at delivery.

## 5 · Modeling operations agents actually use

- **Booleans** (`modifier type='BOOLEAN'`): union/difference/intersect. Coincident faces z-fight — offset cutting geometry by a hair (0.001 mm) or intersect decisively. Apply the modifier before export.
- **Bevel** (`modifier type='BEVEL'`): hardness on edges; segments 1–2 for mechanical, 3+ where highlights must roll. Bevel weight / angle limit controls which edges break — the anti-soap rule.
- **Mirror** with clipping for symmetric parts; **Solidify** for shells from surfaces (wall thickness = the parameter); **Subdivision** only on quad-dominant meshes; **Decimate** to budget (collapse ratio, keep normals; check the volume delta after).
- **Shading**: mark sharp edges (`shade_smooth_by_angle` or edge sharp flags) — smooth-everything is slop; hard-everything is faceted. Mechanical = angle-based; organic = smooth + a few creased edges.
- **Sculpting exists but is not agent-lane** — sculpt data via scripts is impractical; for organic detail, route to generation APIs or displacement (noise bump → displace with real amplitude and re-check the silhouette).

## 6 · Materials — PBR without ceremony

- Principled BSDF is the one node that survives every export path (glTF profile: base color, metallic, roughness, normal — that's the contract that round-trips).
- glTF export maps: `Base Color` → baseColorTexture, `Metallic`/`Roughness` → metallicRoughnessTexture (packed: G=roughness, B=metallic — a custom shader that "works in Blender" does NOT export), normal map node for normalTexture.
- **Bake HIGH→LOW** when detail must survive decimation: sculpt/detail mesh → bake normal + AO to the low-poly's UVs (`bake_maps` pattern: select HIGH then LOW, bake types NORMAL + AO, margin 4–16 px). The high poly never ships; the low poly carries the illusion.
- Texture size matched to budget (craft.md §3). Package textures as PNG for delivery, KTX2 for web runtime (`gltf-transform` does the conversion — formats.md).

## 7 · Export and the roundtrip

- Export ops: `bpy.ops.export_scene.gltf(filepath=..., export_format='GLB', ...)`, `.fbx(...)`, `.obj(...)`, `.stl(...)`. Set `use_selection` when the scene has scaffolding.
- glTF gotchas: Y-up conversion is automatic; apply modifiers with `export_apply=True` or the booleans/bevels vanish; `export_yup` default is right; check `export_materials` packs textures into the GLB.
- **Write a manifest beside the export** (`asset-manifest.json`): source file, phase scripts used, face/vert counts, materials, texture list, units, author date. The manifest is what the next agent reads instead of guessing.
- **The roundtrip (non-negotiable)**: open a FACTORY-STARTUP Blender, import the exported file, print what arrived (objects, faces, UV layers, materials, bounding box), render one clay frame. Import is the only honest test of export — "the file exists" is not a state.

```python
# roundtrip.py — the whole gate in one script
import bpy, json
bpy.ops.wm.read_factory_settings(use_empty=True)
bpy.ops.import_scene.gltf(filepath="delivery.glb")
report = {"objects": len(bpy.data.objects), "meshes": len(bpy.data.meshes)}
for ob in bpy.data.objects:
    if ob.type == 'MESH':
        r = {ob.name: {"faces": len(ob.data.polygons),
                        "uvs": bool(ob.data.uv_layers),
                        "materials": len(ob.data.materials)}}
        report.update(r)
print(json.dumps(report, indent=2))
```

## 8 · Role routing (arjun988's specialist map, condensed)

When the task is big, route by role the way the 94-skill Blender collections do: **modeler** (blockout→forms→detail), **character-artist** (nine-view refs, face/hair, scope discipline), **environment-artist** (layout, kit reuse, scatter with fixed seeds), **asset-optimization** (decimate, bake, LOD ladder), **export-pipeline** (formats, manifests, roundtrip), **camera/cinematography** (only when rendering is the deliverable). One asset, one role at a time — mixing them in one pass is how phases get skipped.

## 9 · Blender as the universal converter

Blender headless is the answer to "convert X to Y" when no direct converter is trustworthy: `--background --python convert.py -- --in part.fbx --out part.glb`. Import (any format) → clean scene of scaffolding → apply transforms → export (any format). For pure web-optimization chains (no DCC features needed), `gltf-transform` CLI is lighter and scriptable without Blender (formats.md §5). Rule: **format conversion with structural edits → Blender; glTF-baseline optimization → gltf-transform; validation → trimesh.**
