# threed

A big skill for **3D model creation** — procedural, Blender-scripted, and AI-generated geometry that actually validates. Born from merging the best 3D-agent skills found on GitHub into one discipline: the mesh is data, the agent's blindness is compensated with two verification channels (numbers + renders), and nothing ships without a roundtrip.

```
An AI writing a 3D model is a blind sculptor. The craft is building
the eyes: gates that measure the mesh, and renders that show it.
```

## What's inside

| Piece | What it does |
| --- | --- |
| **Mesh craft** (`references/craft.md`) | The 70%: the five mesh properties (manifold, orientation, scale, topology, budget), units discipline (STL unitless, glTF meters, drift between steps), budget table per destination, the two-channel verification loop, reference discipline, the 14-item slop death list |
| **Blender lane** (`references/blender.md`) | Headless bpy (binary resolution, factory-startup), the numbered idempotent build-script pattern, review renders (clay/silhouette/wire/checker/turntable/reference-matched), booleans-bevel-bake without slop, export manifests + roundtrip, role routing |
| **Parametric CAD** (`references/cad.md`) | OpenSCAD CSG ($fn discipline, boolean-avoidance rules, minkowski vs hull) and CadQuery/build123d (selectors, fillets, shells) — every dimension a named parameter; tolerances regenerate, never re-draw |
| **Formats & repair** (`references/formats.md`) | The format-contract table (what each format silently drops), trimesh validation/repair pipeline (correct ORDER), manifold3d booleans, gltf-transform web optimization (draco/meshopt/KTX2 — bytes vs triangles), pymeshlab deep repair, delivery manifests |
| **AI generation** (`references/generative.md`) | The sourcing ladder, concept-image-first, price-before-spend with approval gates, the staged game-ready chain (geometry → UV → texture → rig) with drift re-verification, draft-intake repair, the clay-render orbit route (7–53% per-angle drift, measured) |
| **3D printing** (`references/printing.md`) | Printability gates (walls, overhangs, holes, bed fit), orientation logic (layer lines vs load), tolerances that make parts fit, 3MF-over-STL, the user-starts-the-print rule, the print-readiness report |
| **Recipes** (`references/recipes.md`) | 14 dual-stack builds (R01–R14): parametric box/bracket/nameplate/flange, procedural rock/crystal, game crate pipeline, web hero optimization, AI-mesh repair intake, decimation with proof, universal conversion, the mesh audit, the turntable contact sheet |

Plus a Cursor rules export at `exports/threed.mdc`.

## Install

**Claude Code / GLM / any SKILL.md host:**
```bash
git clone https://github.com/lucasgabrieldevgg/threed.git ~/.claude/skills/threed
```

**Cursor:** copy `exports/threed.mdc` to `.cursor/rules/threed.mdc`.

**ClawHub / skills.sh:** the folder is standard skill layout — publish as-is.

## The method (why craft is 70%)

The default failure mode of AI-generated 3D mirrors AI code that never ran: plausible files that break on contact — non-manifold edges, inverted normals, a "30 cm" part at 30 meters. So craft comes first: units declared before building, silhouette proven before detail, every model gated by numbers (trimesh: watertight, winding, volume, extents, budget) and by renders the agent actually inspects (clay, turntable, reference-matched camera). The tool lanes — CAD code, Blender scripts, generation APIs — are just three ways to produce geometry that then faces the same gates. Generation output is a draft; exports get roundtrips; the print button belongs to the user.

## Verification

3D tools move slower than web packages but drift anyway. Versions in `references/` (trimesh 5.1.1, manifold3d 3.5.4, CadQuery 2.8.0, build123d 0.13.0, pygltflib 1.16.5, pymeshlab 2025.7.post1, @gltf-transform 4.5.1, three 0.186.1; Blender 5.2/4.5 LTS lines) were live-checked against PyPI/npm/blender.org on 2026-10-15. Generation-API price anchors are author-time observations — re-verify before promising costs. Blender 5.x API changes are flagged where they bite.

## Attribution

`threed` is an original synthesis (MIT) merging upstream material from `achimala/dream-loop` (MIT), `scenario-labs/skills` (MIT), `majidmanzarpour/blender-game-skills` (MIT), `heyixuan2/bambu-studio-ai` (MIT), `arjun988/blender-skills` (MIT), `viettranx/3dviz-pro-max` (MIT), `samyost1/3dicon` (MIT), `icesixgod/awesome-astra-blender-characters` (MIT), and `RobLe3/cc-blender-skill` (MIT), with concept-only credit to `flowful-ai/cad-skill` (PolyForm NC — not merged). All recipes and the OpenSCAD/CadQuery patterns are original. See [`ATTRIBUTION.md`](ATTRIBUTION.md).

## License

MIT for this repo's original content. Upstream material retains its own licenses — see ATTRIBUTION.md.
