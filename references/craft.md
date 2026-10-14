# Mesh craft — the fundamentals that decide everything else

*Merged from: majidmanzarpour/blender-game-skills (gated phases, silhouette-before-detail, validate-with-numbers), scenario-labs/skills (budget tiers, drift facts), heyixuan2/bambu-studio-ai (units discipline), achimala/dream-loop (target-image loop), icesixgod/awesome-astra-blender-characters (multi-view reference discipline), viettranx/3dviz-pro-max (verify-what-appears ethos). Package versions live-verified 2026-10-15.*

The default failure mode of AI-generated 3D is identical to AI-generated code that never ran: plausible-looking files that fall apart on contact — non-manifold edges, inverted normals, a "table" 400 units wide in a scene that expects meters. Craft here means internalizing the properties that make geometry *valid*, before any tool touches it.

## 1 · The mesh is data — the five properties

Every model, whatever the tool, carries five properties that gates check:

| Property | Meaning | Gate |
| --- | --- | --- |
| **Manifold / watertight** | Every edge shared by exactly 2 faces; encloses a volume | `is_watertight`, edge-face count |
| **Orientation** | All normals point outward; signed volume > 0 | `is_winding_consistent`, volume sign |
| **Scale / units** | Bounding box matches intent in real units | `extents` vs declared size |
| **Topology** | Tri/quad structure, no degenerate faces, no duplicate verts | face/vert audit, zero-area check |
| **Budget** | Face count fits the destination | count vs budget table |

A model that fails any of these is not "mostly done" — it is broken in a way that surfaces later (slicer errors, dark shading, physics explosions, engine import warnings). The three manifestations agents actually hit:

- **Non-manifold** → slicers refuse or misheal it; boolean operations fail; simulations blow up.
- **Inverted normals** → faces render black or inside-out; a later pipeline step inherits the flip.
- **Unitless scale** → the classic: an AI-generated "30 cm" part arrives as a 30-meter mesh, or STL's unitless numbers meet a slicer assuming millimeters.

## 2 · Units and scale — the #1 failure

- **Declare units in the intent, before building.** "80 mm tall", "1.8 m character", "fits a 100 mm plate". If the user didn't say, ask — generation and modeling both cost effort that the wrong scale wastes.
- **STL is unitless** (the number is "units"; slicers almost always assume mm). **OBJ is unitless** by spec. **glTF is meters, always.** **3MF carries units.** **FBX has a unit setting that imports respect inconsistently** — verify after import, every time.
- **Generation APIs return unit-normalized meshes** (longest side = 1.0) with real scale optionally in a node transform — the API's reported `dimensions` can read normalized even when the download is correct. Trust the imported bounding box, not the API's self-report.
- **Scale drifts between pipeline steps.** Generation → unwrap → retexture → rig can shift scale and pivot *within one vendor's chain*. Re-measure the bounding box after every stage, not just at the end.

The scale gate, run on everything:

```python
import trimesh
m = trimesh.load("model.glb", force="mesh")   # or "scene", then concatenate
print("extents:", m.extents)          # compare against intent, in the file's units
print("bounds:",  m.bounds)           # min/max corners — is the origin where you think?
```

## 3 · Budgets — the destination decides

| Destination | Budget (faces) | Notes |
| --- | --- | --- |
| Game-ready hero prop / character | 10k–50k tris | Generation APIs expose `faceBudget`/`faceLimit` — set it at generation, or decimate after |
| Game background / environment piece | 1k–10k tris | LOD ladder: distance decides |
| Web hero object (three.js) | < 100k tris | Draco/meshopt compresses bytes, NOT render cost — the count is still paid per frame |
| Web icon / small embed | 2k–10k tris | 4K textures on a 128 px asset is slop |
| 3D print (FDM) | No hard limit | Practical: < 500k tris or the slicer crawls |
| AR (USDZ) | < 100k tris, textures < 2K | Apple's quick-look comfort zone |

Texture budget follows the same logic: match texture resolution to on-screen size, not to "quality" vibes. A 128 px game asset with 4K PBR maps is a defect, not luxury.

## 4 · Topology — tris, quads, and the truth

- **glTF/GLB, OBJ, STL store triangles.** Quads exist in FBX and DCC-native formats. Converting a quad mesh to GLB triangulates it — irreversibly for that file. Keep the quad FBX beside the GLB if quads matter (rigging targets, subdivision).
- **Quad topology matters for** subdivision surfaces and deformation (rigging). For static props, engine import, and printing, tris are fine.
- **Degenerate geometry audit** — the silent killer: zero-area faces (three collinear verts), duplicate vertices (unmerged seams), floating unused verts, internal faces (geometry hidden inside the shell). All caught programmatically:

```python
import trimesh, numpy as np
m = trimesh.load("part.stl")
areas = m.area_faces
print("zero-area faces:", int((areas < 1e-12).sum()))
print("duplicate verts:", len(m.vertices) - len(m.unique_vertices()))
print("watertight:", m.is_watertight, "| winding consistent:", m.is_winding_consistent)
print("volume:", m.volume, "(positive = outward normals, usually)")
if not m.is_winding_consistent or m.volume < 0:
    m.fix_normals()
```

- **Fix order matters**: merge vertices first (`merge_vertices()`), then repair normals (`fix_normals()`), then re-check watertight. Fixing normals on split verts re-bakes the error.

## 5 · Origin, pivot, orientation

- The **origin** is the model's anchor: how it places in a scene, how it rotates, where it attaches. Decide it deliberately — center-of-mass for props, feet-on-origin for characters (engine-ready), build-plate-contact for prints.
- **Z-up vs Y-up**: Blender/CAD are Z-up; glTF and most engines are Y-up. Exporters convert, but a manual rotation baked into vertices instead of the transform is a classic silent bug.
- A transform baked into the mesh (`matrix_world` applied) cannot be un-done cheaply later; a transform left on the object is data. Prefer leaving placement on the object and exporting with "apply transforms" only at delivery.

## 6 · The two-channel verification loop

An agent cannot "look" at a mesh in its head — so verification is engineered, not eyeballed:

**Channel 1 — numbers (the gates):**

```python
import trimesh
m = trimesh.load("delivery.glb", force="mesh")
report = {
    "watertight": m.is_watertight,
    "winding_consistent": m.is_winding_consistent,
    "volume_positive": m.volume > 0,
    "extents": m.extents.tolist(),
    "faces": len(m.faces),
    "verts": len(m.vertices),
}
assert report["watertight"] and report["winding_consistent"] and report["volume_positive"]
```

**Channel 2 — renders (the eyes you borrow):** render the model from fixed views — front, side, ¾, top — as clay (flat gray, no texture) and look at the images. Agents that can read images must actually read them: dark regions = inverted normals; a silhouette gap = a hole; zebra striping = z-fighting; floating specks = orphan geometry. A turntable (16 cameras, 22.5° apart) is the honest view — every hidden face gets its frame. This is the freeze-frame discipline of motion, applied to geometry: the render is the proof, and "the script exited 0" is not.

**The gate protocol (after every build phase):**

1. Run the numbers gate — hard fail on any of the five properties.
2. Render the fixed views — inspect for the visual tells above.
3. Compare against intent: bounding box vs declared size, silhouette vs reference.
4. Only a model that passes all three moves to the next phase. Detail added on a failed silhouette is rework scheduled.

## 7 · Reference discipline

- **Never model from a filename or a description when an image exists.** Look at the image first — read the actual silhouette, proportions, and parts.
- **A single reference hides three sides.** State what is inferred: "back and underside inferred from typical construction" — never present a guessed back as fact.
- **For characters, generate a multi-view reference set first** (the astra discipline): 8 horizontal views at 45° spacing plus top — each generated from the SAME original image and feature constraints, orthographic, same scale and camera height — then check the set for mutual consistency before any modeling. Conflicts resolve toward the original image; generated views are design aids, not geometric truth.
- **The target-image loop (dream-loop):** when fidelity to a look is the goal, generate or receive the "dream version" target FIRST — a real in-engine-style screenshot, not concept art — then iterate the build until the render matches the target. Compare from the same camera, at the same pixel size.
- **Video beats still for timing references; stills beat nothing for proportion references.**

## 8 · The 3D-slop death list

Cut on sight, in review:

1. **Non-manifold mesh shipped as "done"** — it will break in the slicer/engine/boolean.
2. **Inverted normals** — dark or inside-out faces; fix_normals before ANY export.
3. **Unitless or wrong-scale model** — "it's 100 units tall": units of what?
4. **Detail before silhouette** — ornament on wrong proportions; scrap with filigree.
5. **Everything smooth-shaded** with no hard edges marked — mechanical parts look like soap.
6. **The export that was never reimported** — "I wrote the GLB" without a roundtrip.
7. **Z-fighting coincident surfaces** — two faces at the same plane flicker; offset by a real tolerance or fix the CSG.
8. **One blob where the spec said parts** — a "table" that is a single fused mesh cannot have its top replaced.
9. **Placeholder dimensions** — "about 10 cm" never confirmed with the user.
10. **4K textures on a 128 px asset** — budget mismatch, wasted bytes and memory.
11. **Quad topology promised in a GLB** — impossible; quads live in FBX/DCC only.
12. **Unverified repair** — "I ran a repair script" without re-running the gates; repair can delete real geometry (check the volume delta before/after).
13. **Textureless "PBR"** — a single flat color material labeled "PBR" with no roughness/metallic variation.
14. **The viewport claim** — "it looks fine in the viewport": the viewport is not a render, and nobody looked anyway.

## 9 · Reproducibility

- Builds are **scripts, not hand edits** — numbered phases (`00_setup.py`, `10_blockout.py`, `20_detail.py`…), each idempotent: open the master, delete what this phase owns, rebuild it, save. Rerunning any phase is safe; the build is replayable from source.
- Constants are real measurements with comments naming where they came from — `WALL = 2.4  # mm, from calipers 2026-10-15` or `# inferred (reference shows front only)`.
- Seeds fixed for anything randomized (scatter, noise, variation) — same build, same model, every time.
