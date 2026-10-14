# Recipes — ready-to-adapt builds, dual-stack

*Original implementations for this repo (MIT), merging technique ideas from upstream: majidmanzarpour/blender-game-skills (gated phases + roundtrip in R07/R12), heyixuan2/bambu-studio-ai (analyze-intake in R10), achimala/dream-loop (cost-gate in R09), samyost1/3dicon (R09 pattern), scenario-labs/skills (R08 optimization lane). Full credits: ATTRIBUTION.md. Every recipe ends in the same place: the numbers gate + a render you actually look at.*

Conventions: CAD recipes are OpenSCAD; mesh recipes are Python (trimesh or bpy headless); every recipe states its budget and gates. Constants are parameters — adapt, don't transcribe blindly.

## R01 · Parametric box with lid (CAD)

The enclosure workhorse. OpenSCAD, tolerance as parameter:

```scad
wall = 2.4; inner = [58, 38, 22]; gap = 0.25; $fn = 48;
module shell() {
    difference() {
        minkowski() { cube(inner); sphere(wall); }
        translate([wall, wall, wall]) cube(inner);
    }
}
module lid() {
    translate([0, 0, wall + inner.z + gap])
        linear_extrude(wall) offset(r = wall) square([inner.x, inner.y]);
}
shell(); lid();
```

Gates: watertight, walls ≥ 2.4, lid gap 0.25 ± 0.05 on FDM (measure your printer, regenerate). Export both parts; label which is which in the manifest (multi-part STL confuses slicers — prefer separate files or 3MF parts).

## R02 · L-bracket with bolt holes (CAD)

cad.md's worked example: sketch → extrude → fillet → holes at `d = bolt + 0.2`. Gates: hole diameter calipers-check on first print; strength orientation noted (flat on back). CadQuery when fillets/chamfers matter, OpenSCAD when pure CSG suffices.

## R03 · Embossed nameplate / keychain (CAD)

```scad
text("MARINA", size = 12, font = "Liberation Sans:style=Bold")
```

`linear_extrude(2)` the text over a `hull()`-rounded plate; punch a `teardrop`-modded hole (circle + rotated square union) ≥ 3 mm for a ring. Gates: emboss depth ≥ 0.6 mm per layer; hole horizontal or vertical, never diagonal.

## R04 · Flange / pipe adapter (CAD)

Two `rotate_extrude`d profiles or `cylinder` unions; bolt ring via `for (a = [0 : 360/n : 360]) rotate(a)` placement. The tolerance lesson: hub fits get +0.2, seal faces get 0. Gates: concentricity (extents symmetric), bolt circle count/spacing vs the mating part — ask for the mating dims, don't guess.

## R05 · Low-poly rock (mesh, procedural)

Deterministic scatter-and-noise, seed fixed:

```python
import trimesh, numpy as np
rng = np.random.default_rng(42)
m = trimesh.creation.icosphere(subdivisions=1, radius=1.0)
for _ in range(3):                        # displace along normals, quantized = facets
    d = rng.normal(size=(len(m.vertices), 1)) * 0.18
    m.vertices += m.vertex_normals * d
m.vertices -= m.vertices.mean(axis=0)
m.export("rock.stl")
```

`subdivisions=1` + quantized displacement = the faceted stylized look (80 faces, game-background budget). Gates: watertight (it stays watertight under normal displacement — verify anyway), scale to cm at export.

## R06 · Crystal / gem (mesh, Blender)

Cone + `cast` modifier (factor 0.2) or hand-built: `bpy.ops.mesh.primitive_cone_add(vertices=6)` → scale → tip at origin → flat shading (`shade_flat`) — flat shading IS the material for gems. Export GLB with a simple Principled (transmission if engine supports it). Gates: flat shading survives export; budget < 200 tris.

## R07 · Game-ready crate (mesh, Blender, full pipeline)

The whole blender.md lane in one recipe: `00_init` (units m, ref cube) → `10_blockout` (1.0 m³ cube primitive) → `20_forms` (bevel 0.02, edge loops at plank seams) → `30_detail` (normal-map greebles instead of geometry: bake or generate) → `40_materials` (Principled: wood albedo, roughness map) → `50_export` (GLB, apply modifiers, manifest) → roundtrip reimport. Budget: 500–2000 tris. Gates: silhouette render vs reference at gameplay px; UV checker pass; roundtrip report.

## R08 · Web hero object (mesh → optimize)

Any decent mesh → web budget: decimate to < 100k tris → `gltf-transform` dedup/weld → draco `--quantize-position 14` → textures to WebP ≤ 1024 (or KTX2 if the runtime wires `KTX2Loader`). Deliver GLB + manifest noting the decoder the consuming three.js code must register. Gates: before/after Hausdorff-ish check (volume delta < 5%), one clay render.

## R09 · 3D icon (mesh, small-object lane)

The 3dicon pattern condensed: ONE design (still or tiny mesh), user approves, then the pipeline — 2–10k tris, single material, transparent-friendly (alpha in textures, not geometry), turntable contact sheet for review, WebP/GIF preview beside the GLB. Cost-gate: approval after the still, before any motion/extra views.

## R10 · Repair an AI-generated mesh (the intake)

The generative.md §5 pipeline as one script:

```python
import trimesh
m = trimesh.load("draft.glb", force="scene")
parts = {name: g for name, g in m.geometry.items()}
keep = [g for g in parts.values()
        if len(g.faces) > 500 and g.is_watertight is not None]      # drop junk shells
mesh = trimesh.util.concatenate(keep) if len(keep) > 1 else keep[0]
v0 = mesh.volume
mesh.merge_vertices(); mesh.remove_degenerate_faces(); mesh.fix_normals()
print(f"shells dropped: {len(parts) - len(keep)} | volume: {v0:.0f} -> {mesh.volume:.0f}")
assert mesh.volume > v0 * 0.95, "repair amputated geometry — inspect shells manually"
mesh.export("repaired.glb")
```

Then decimate to budget, re-gate, render. The assert is the recipe: repair that deletes 5%+ of volume needs eyes, not trust.

## R11 · Decimate to budget with quality proof

`simplify_quadric_decimation(budget)` → volume delta + silhouette render before/after → manifest records the cost. Never decimate a rig target (skin weights die with the verts).

## R12 · Universal format conversion (Blender headless)

```python
# convert.py — blender --background --factory-startup --python convert.py -- --in x.fbx --out x.glb
import bpy, sys
argv = sys.argv[sys.argv.index("--") + 1:]
bpy.ops.import_scene.fbx(filepath=argv[argv.index("--in") + 1])
bpy.ops.export_scene.gltf(filepath=argv[argv.index("--out") + 1],
                          export_format="GLB", export_apply=True)
```

Swap import/export ops per format pair. Post-convert: re-gate + check what the target format dropped (formats.md §1). For GLB→GLB optimization, gltf-transform instead (no Blender needed).

## R13 · The mesh audit (run this first, on everything)

craft.md §6's gate as a standalone: per-object watertight/winding/faces/extents report, exit 1 on failure. The universal first step of ANY 3D task — audit before you touch.

## R14 · Turntable contact sheet (the eyes)

Blender headless: 16 cameras at 22.5°, even height, Workbench clay render, tile 4×4 into one PNG (Pillow), look at it. The single highest-value verification in the whole skill — every hidden face gets a frame, every inverted normal a dark patch, every floating shell a speck.
