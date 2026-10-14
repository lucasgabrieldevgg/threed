# Formats, conversion, and programmatic repair

*Merged from: scenario-labs/skills (format contracts, quad/tri truth, staged-chain formats), majidmanzarpour/blender-game-skills (manifest + roundtrip), viettranx/3dviz-pro-max (capture-verify ethos), heyixuan2/bambu-studio-ai (analyze/repair lane). trimesh 5.1.1, manifold3d 3.5.4, pygltflib 1.16.5, numpy-stl 4.0.1, pymeshlab 2025.7.post1 (PyPI) and @gltf-transform 4.5.1 (npm) live-verified 2026-10-15.*

Formats are contracts, not containers. Picking the wrong one doesn't error — it silently drops the thing the contract didn't describe (units, quads, materials, hierarchy), and the loss surfaces one pipeline step later as "the model looks wrong".

## 1 · The format table

| Format | Units | Topology | Materials | Carries | Use for |
| --- | --- | --- | --- | --- | --- |
| **GLB / glTF** | meters (in spec; exporters convert) | triangles only | PBR metal-rough (full) | hierarchy, animations, skins, lights, cameras | Web, engines with glTF import, interchange of finished assets |
| **FBX** | unit setting (usually cm) | tris AND quads | Phong/PBR-ish, engine-dependent | rigs, skins, blendshapes | Rigged/quad assets for Unity/Unreal/DCC handoff |
| **OBJ** | none (convention: whatever) | tris (+ quads loosely) | MTL sidecar (Phong-era) | UVs, groups | Raw geometry interchange; almost no semantics |
| **STL** | none (slicer assumes mm) | triangles only | none | nothing but triangles | 3D print input; the lowest common denominator |
| **3MF** | mm (explicit) | triangles | colors, parts, print metadata | units!, multiple objects | Modern print delivery — STL + truth |
| **PLY** | none | tris/quads | vertex colors | dense data, scans | Point clouds, scan output |
| **USDZ / USDA** | meters | tris | PBR | AR anchors, layers | Apple AR / Omniverse |
| **STEP** | mm (real) | B-rep (not a mesh!) | none | exact geometry, history-lite | CAD interchange — engineers re-open it |
| **.blend** | scene setting | any | any | EVERYTHING (version-locked) | Source of truth only — never an interchange format |

The two truths that bite agents most: **quads only survive in FBX/DCC** (glTF triangulates on write — a GLB cannot carry quads, period), and **STL/OBJ are unitless** (the number means nothing until a consumer decides mm).

## 2 · trimesh — the numbers lane (Python)

The validation/repair workhorse. `force="mesh"` concatenates a scene into one mesh (report per-object first — concatenating THEN checking hides which object failed):

```python
import trimesh

# --- audit: per object, the full gate report ---
scene = trimesh.load("model.glb")          # GLB loads as a Scene
for name, geom in scene.geometry.items():
    print(f"{name}: watertight={geom.is_watertight} "
          f"winding={geom.is_winding_consistent} faces={len(geom.faces)} "
          f"extents={geom.extents.tolist()}")
    if not geom.is_winding_consistent or geom.volume < 0:
        geom.fix_normals()

# --- repair: the standard pipeline, in order ---
m = trimesh.load("draft.stl")
m.merge_vertices()          # 1. weld seams first (never fix normals on split verts)
m.update_faces(m.unique_faces())   # 2. drop duplicate faces
m.remove_degenerate_faces() # 3. zero-area faces
m.remove_infinite_values()
m.fix_normals()             # 4. THEN orientation
m.fill_holes()              # 5. small boundary loops (big holes need real modeling)
m.process(validate=True)    # 6. re-merge everything consistently
print(m.is_watertight, len(m.faces))       # re-run the gate — repair can delete geometry
# volume delta check: if volume dropped >5% after "repair", it amputated something real
```

- **Boolean via manifold3d**: `trimesh.boolean.union([a, b], engine="manifold")` — exact, robust booleans on watertight meshes (the engine behind CadQuery 2's kernel). Non-watertight input → repair first or the boolean lies.
- **Decimation to budget**: `mesh.simplify_quadric_decimation(face_count)` (needs `fast_simplification`/`open3d` backend depending on version) — then CHECK: volume delta < 5%, silhouette render still passes. Decimate → re-gate, never decimate → trust.
- **Cross-sections / measurements**: `mesh.section(plane_origin, plane_normal)` → 2D polygon → wall thickness and area checks without any GUI (the printing lane's measuring stick).

## 3 · gltf-transform — the web-optimization lane (Node)

```bash
npx @gltf-transform/cli optimize in.glb out.glb --compress draco --texture-compress webp
# or surgical passes:
npx @gltf-transform/cli dedup in.glb out.glb            # shared materials/textures
npx @gltf-transform/cli prune --keep-extras in.glb out.glb
npx @gltf-transform/cli weld in.glb out.glb             # index → welded verts
npx @gltf-transform/cli draco --quantize-position 14 in.glb out.glb
```

- **Draco vs meshopt**: both compress geometry bytes; meshopt (EXT_meshopt_compression) decodes faster on the GPU path (three.js `MeshoptDecoder`), Draco squeezes harder. Either way: **compression shrinks bytes, not triangles** — the 500k-tri mesh still renders 500k tris; decimate first, compress second.
- **Texture lane**: WebP/AVIF via `texture-compress`, KTX2 (Basis) for GPU-native mipped textures in production three.js — `ktx2` command handles the UASTC/ETC1S choice (UASTC for normals/quality, ETC1S for albedo/size).
- **Quantize-position 14** ≈ sub-0.01 mm at meter scale — visually lossless, big savings. 12 and below starts eating fine detail on large scenes.
- three.js loads Draco/meshopt/KTX2 via addons (`DRACOLoader`, `MeshoptDecoder`, `KTX2Loader`) — the consuming code must wire the decoder or the file won't open; note it in the delivery manifest.

## 4 · pymeshlab — when trimesh isn't enough

Scan-scale repair and remeshing: `pymeshlab` (2025.7) wraps MeshLab's filters — the deep-repair lane for disaster meshes: `compute_texcoord_parametrization`, `repair_non_manifold_edges_by_removing_faces`, `surface_distance` (which Hausdorff distances between two meshes — the honest "how much did decimation cost me" metric, better than volume alone), resampling/remeshing filters. Use it when the mesh comes from photogrammetry/generation and has the pathology catalog (islands, non-manifold edges by the thousand, self-intersections).

## 5 · The conversion ladder

| Job | Tool |
| --- | --- |
| Any-format → any-format, with structural edits | Blender headless (blender.md §9) |
| GLB → optimized GLB (web) | gltf-transform CLI |
| Mesh → mesh validation/repair | trimesh (+manifold3d) |
| Mesh → heavy repair / remesh / distances | pymeshlab |
| CAD → mesh (STEP→STL) | CadQuery exporters / OpenSCAD render |
| Mesh → CAD (reverse) | NOT a real lane — mesh-to-B-rep is approximation; manage expectations |

Conversion is where quads, units, and materials die. After ANY conversion: re-run the numbers gate AND check what the new format dropped (a GLB→OBJ roundtrip loses PBR; an FBX→GLB keeps materials but triangulates; a GLB→STL keeps nothing but triangles).

## 6 · The delivery manifest

Every delivered asset ships with `asset-manifest.json` beside it:

```json
{
  "asset": "treasure-chest.glb",
  "source": "generated (tripo) + repair + decimate",
  "units": "meters (glTF)",
  "bbox_m": [0.48, 0.32, 0.36],
  "faces": 24310, "verts": 12887,
  "materials": 3, "textures": ["albedo.webp 1024", "normal.webp 1024"],
  "compression": "draco, quantize-position 14",
  "validation": {"watertight": true, "winding": true, "volume_positive": true},
  "roundtrip": {"reimported_objects": 1, "uv_layers": 1, "render_check": "review/final.png"},
  "history": ["build/00_init.py", "gen/draft.glb", "repair/fix.py", "build/50_export.py"]
}
```

The manifest is what the next agent (or human) reads instead of re-deriving the truth. Files without manifests are orphans.
