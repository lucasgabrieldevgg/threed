# Attribution & Sources

`threed` is an original synthesis commissioned for this repo: the craft core (validation gates, units discipline, slop list) and the routing structure were merged and rewritten from upstream skills; all recipes in `references/recipes.md` and the OpenSCAD/CadQuery patterns in `references/cad.md` were authored original for this repo (no licensed upstream existed for parametric CAD — see Surveyed below); package versions were live-verified against PyPI and npm (15 October 2026). The synthesis, structure, recipe code, and merged rule set are MIT. Upstream material retains its original license.

## Upstream skills (merged)

| Source | What was merged | License |
| --- | --- | --- |
| `achimala/dream-loop` | The target-image loop (build to a rendered target, compare from the same camera), the asset-sourcing ladder (existing → image-to-3D API → Blender → procedural, with permission rules), the per-asset image-extraction trick for generation inputs, image-generated textures/normals over procedural noise, price anchors (Tripo-class ≈ $0.30, Trellis-class ≈ $0.02 per asset — author-time, re-verify), time-budget ethics (no fidelity degradation to hit budgets) | MIT |
| `scenario-labs/skills` (scenario-3d, scenario-orbit-views, and related) | Concept-image-first generation (image-to-3D over text-to-3D for art direction), price-before-spend (dry-run/quote before paid runs, batch math, jobs_wait batching), the staged game-ready chain (geometry → UV unwrap → retexture → rig) with per-step re-verification, scale/pivot/normal drift facts, unit-normalized API meshes vs node transforms, quad-topology-in-FBX-only and glTF-triangulates truths, faceBudget 10k–50k game-ready, T/A-pose concept for riggable characters, multi-view/3d23d refinement concepts, the clay-render-per-camera orbit route with the 7–53% per-angle drift finding, delight/unlit material notes | MIT |
| `majidmanzarpour/blender-game-skills` (blender-image-to-3d) | The gated-phase build protocol (rendered evidence at every gate, matched to the reference camera at gameplay pixel size), silhouette-before-detail ("most wasted work is detail added on wrong proportions"), numbered idempotent build scripts (NN_phase.py, delete-what-you-own), measured-comparison review sheets, validate-with-exit-1 topology/transform/UV/budget gates, bake HIGH→LOW normal/AO workflow, export manifest + roundtrip reimport check, Blender binary resolution order and headless factory-startup pattern, Blender 5.x API-change caution | MIT |
| `heyixuan2/bambu-studio-ai` | Printing ground rules (the user starts the print; analyze every model whatever its source; preview before slicer; know the size before generating — "how big? e.g. 80 mm tall"; search-first on MakerWorld/Printables), orientation policy (re-orient AI/downloaded models, never parametric ones), the analyze-repair lane (wrong units, floating parts, thin walls, plate fit), 3MF-over-STL for multi-part/color delivery | MIT |
| `arjun988/blender-skills` | The specialist role-routing map (modeler / character-artist / environment-artist / asset-optimization / export-pipeline / camera-cinematography) condensed into blender.md §8; the collection's structure informed the "one asset, one role at a time" discipline | MIT |
| `viettranx/3dviz-pro-max` | The verify-what-appears-on-screen ethos (capture and inspect rendered frames instead of describing them), factual-claims-need-sources discipline, and the tiered ambition framing for web 3D scenes (its Three.js scene recipes are out of scope here — the `motion` skill owns web animation/interaction territory) | MIT |
| `samyost1/3dicon` | Cost-gated approval (one design, user approves, then the expensive pipeline — generalized to concept-then-generate in generative.md §2/§3), transparent-asset delivery pattern, "check the keys are present before running — a run that dies halfway has already spent the money" | MIT |
| `icesixgod/awesome-astra-blender-characters` | The nine-view reference discipline (8 horizontal views at 45° spacing + top, generated from the same original image and feature constraints, orthographic, same scale/camera height, cross-checked for consistency), "script success ≠ likeness success", scope discipline (partial-face work is not full-character work; never present a guessed back as fact), version-chain separation | MIT |
| `RobLe3/cc-blender-skill` | Environment-check-first pattern (binary/deps/input verified before any work, fail fast with the fix named), the wireframe/orthographic-drawing input lane, MCP-vs-headless execution notes | MIT |

## Concept-only credits (not merged)

| Source | Concept credited | License |
| --- | --- | --- |
| `flowful-ai/cad-skill` | The parametric 3D-printable generation concept (CadQuery-first, printability-aware generation) informed cad.md's scope and emphasis. No content merged — PolyForm Noncommercial is incompatible with this repo's MIT synthesis | PolyForm NC 1.0.0 |
| `EdwinjJ1/3d-print-skill` | Search-first printing workflow and manifold-safe-output discipline informed printing.md's structure. No content merged | No LICENSE file |

## Surveyed, excluded (documented for transparency)

- `mfranzon/render` (170★, "create beautiful 3D models directly from coding agents") — no LICENSE file; excluded per house rule.
- `iancanderson/openscad-agent` (131★) — no LICENSE file; excluded. (Its territory — OpenSCAD for printable designs — is covered in `cad.md` with original content.)
- `kevinbadi/blender-skills` (155★) — no LICENSE file; excluded.
- `BaLaurent/ClawdCAD` (19★, OpenSCAD) — GPL-3.0 copyleft; excluded from merge.
- `codeofaxel/Kiln` (92★) — AGPL-3.0 MCP server, printing; excluded.
- `rehan-remade/universal-modder` (7624★, MIT) — game modding; out of scope.
- `freshtechbro/claudedesignskills` (1053★, MIT) — web 3D graphics/animation; `motion` territory.
- `iart-ai/webgl-animation-skills` (21★, MIT) — WebGL motion; `motion` territory (same upstream family as motion's merge).
- `Impertio-Studio/Three.js-Claude-Skill-Package`, `tsogjavklann/awwwards-3d`, `MustBeSimo/web-design-studio` — web Three.js scene/site skills; out of scope.
- `NeonGlay/inventor-mcp` (35★, MIT) — Autodesk Inventor MCP; proprietary-CAD lane, concept noted.
- `NVIDIA-Omniverse/usd-convert-cad` (30★) — CAD→OpenUSD conversion; custom license, niche lane.
- `HRuiCcc/RuiC-card-skill`, `SyphoDev-yt/roblox-ai-skills`, `ig-shadow-walker/BlenderXAlpha-3DGenSkill`, `k-telux/OpticalModeler`, `EGSECDA/vocaloid-style-mv-pipeline`, `op7418/guizang-sports-skill`, `SteadfastAsArt/geoscience-skills`, `sunxiayi/make-blender-education-video-skill`, `OpenCesium/Cesium-Skills`, ` dcc-mcp/dcc-mcp-core`, `moellere/WireStudio` — out of scope (web cards, Roblox, niche verticals, education video, geo, AEC).
- Survey method: 16 GitHub search queries (claude skill 3d / openscad / blender / cadquery / 3d printing / text-to-3d / meshy / tripo / gltf / stl / agent skill 3d modeling / blender skill agent / sdf / claude-skill 3d), 82 unique repositories ranked, 14 cloned and studied in full (2026-10-15).

## Original verification work

- **Package version verification** (2026-10-15): trimesh 5.1.1, manifold3d 3.5.4, cadquery 2.8.0, build123d 0.13.0, pygltflib 1.16.5, numpy-stl 4.0.1, pymeshlab 2025.7.post1 (PyPI); @gltf-transform/core and /cli 4.5.1, three 0.186.1 (npm, consistent with the `motion` skill's verification); Blender 5.2 LTS / 4.5 LTS / 4.2 LTS current lines (blender.org); OpenSCAD 2021.01 stable line (openscad.org/GitHub).
- **License verification**: every merged source's LICENSE read 1:1 from the cloned repo (not from API metadata — the API reported `cad-skill` as merely NOASSERTION; the file is PolyForm NC).
- **Recipe corpus**: all 14 recipes in `references/recipes.md` and the OpenSCAD/CadQuery content in `references/cad.md` are original to this repo — merging technique ideas from upstream (gated phases, repair order, cost gates) with original implementations.
- **The structure, routing, quality gates, and wording** are original to this repo.

## Individual credits

- **Anshu Chimala** (achimala) — dream-loop's target-image loop and the sourcing ladder that orders every generative decision.
- **Scenario** (scenario-labs) — the operating discipline for generation APIs: schema before run, price before spend, staged chains, drift verification.
- **Majid Manzarpour** (majidmanzarpour) — the gated-phase build protocol with rendered evidence; the rigor backbone of blender.md.
- **Yixuan He** / TieGaier (heyixuan2) — bambu-studio-ai's ground rules that make an agent safe around printers.
- **arjun988** — the specialist-role map for Blender work.
- **Viet Tran** (viettranx) — 3dviz-pro-max's verify-what-rendered ethos.
- **Sam Yost** (samyost1) — 3dicon's cost-gated approval pattern.
- **awesome-astra-blender-characters** (icesixgod) — the nine-view reference discipline for character work.
- **RobLe3** — the environment-check-first pattern and wireframe-input lane.
- **flowful-ai** — the parametric-printable concept (credited; content not merged, license incompatible).
- The maintainers of trimesh, manifold3d, CadQuery, build123d, OpenSCAD, glTF-Transform, pymeshlab, and Blender — the tools this skill is useless without.

Any errors in the merged synthesis are this repo's, not theirs.

If you redistribute `threed`, keep this file and the per-file credits with it.
