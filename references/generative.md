# AI-generated 3D — text/image-to-3D as a disciplined pipeline

*Merged from: achimala/dream-loop (sourcing ladder, price anchors, target-extraction trick), scenario-labs/skills (concept-first, schema-before-run, dry-run pricing, staged game-ready chain, drift verification, format contracts), samyost1/3dicon (cost-gated approval — one-then-ask), icesixgod/awesome-astra-blender-characters (multi-view generation discipline), heyixuan2/bambu-studio-ai (know-the-size-first, AI-mesh analysis). Price anchors are author-time observations — re-verify before promising costs.*

Generation APIs give you a textured, near-ready mesh in minutes for cents-to-dollars. They are also the fastest way to ship broken geometry at scale: unit-normalized meshes, non-manifold shells, inverted faces, baked-in lighting, and 800k triangles you didn't ask for. The discipline: **generation output is a DRAFT** — it enters the same gates as everything else, and money is spent in gated stages, never in one heroic run.

## 1 · The sourcing ladder (before any money)

Work top-down; do not skip steps to "save time" — each rung is cheaper than the one below it:

1. **An existing model, if the user allows downloads.** Search MakerWorld/Printables/Sketchfab (license check!) before generating anything. If unspecified, assume downloads are NOT allowed — ask.
2. **Image-to-3D API** — the sweet spot for organic/complex single objects: characters, props, buildings, greenery. Cents per asset.
3. **Blender scripting** (blender.md) — when the shape is definable, or generation fails the silhouette twice.
4. **Pure procedural code** — when geometry is mathematical (parts, patterns) or nothing else is available.

For texturing at any rung: image-generated textures/normal maps beat procedural noise — "do not replace missing generated textures with procedural noise or flat-color substitutes" (dream-loop's rule; texture is most of perceived quality).

## 2 · Concept-first, not text-first

**The reliable pipeline generates a concept image, then feeds it to image-to-3D** — direct text-to-3D exists but the image intermediary gives art-direction control (pose, angle, lighting, palette) that text prompts cannot. The concept image IS the target; the mesh chases it.

- Generate the concept: single centered subject, plain background, the exact view you want modeled.
- **Extract clean per-asset inputs**: pass the target/scene image to image gen and ask for "just the target object on a solid/transparent background" — then feed THAT to image-to-3D, so the mesh matches the target instead of re-imagining it.
- **Multi-view beats single-view**: many models accept 2–8 images of one subject (front first — ordering varies per schema; read the schema). More angles = fewer invented backs.
- **Characters built for rigging get T- or A-pose concepts** (full body, arms clear of torso, plain background) — riggers infer joints from the pose.
- Ask the user to approve the concept BEFORE generation: the concept is cents, the chain is dollars — and every later stage inherits the concept's design (3dicon's one-then-ask rule, generalized).

## 3 · Price before spend — the money discipline

- **Quote/dry-run before every paid call** where the API offers it; where it doesn't, check the pricing page and state the number to the user first.
- **Approval gates between costly stages**: concept → generate → refine are separate budgets; the user signs off at each.
- **Batch math before launching**: one view's quote × N views; one asset's cost × scene's asset count. Ten "30-cent" assets is a $3 run — fine, but say it BEFORE, not in the invoice apology.
- Author-time anchors for orientation (VERIFY CURRENT): strong image-to-3D (Tripo-class) ≈ $0.30/asset; small models (Trellis-class) ≈ $0.02/asset — route: strong for heroes/characters, small for environmental props. Provider families as of 2026-10: Meshy, Tripo, Rodin, Hunyuan3D, Trellis — via their own APIs or aggregators (fal.ai, Scenario, Replicate).
- **Never rebuy the same failure**: one variable per rerun (seed fixed, change the one thing), and regenerate the failing view/part — not the whole asset.

## 4 · The staged chain toward game-ready

One generation run bakes texture onto UVs the generator chose. A mesh headed for an engine goes further, **run by run, each output feeding the next tool**:

```
geometry (texture off, faceBudget set) → UV unwrap → retexture → rig
```

- **Geometry first with `texture: false`** where the schema offers it (cheaper, and texture comes later anyway). Set the face budget AT generation when the parameter exists — 10k–50k tris for game-ready.
- **UV unwrap** on the bare mesh; **retexture** on the unwrapped mesh (prompt/image/style reference); **rig last**, on the topology that ships.
- **Each step re-verified**: normals, scale, pivot, and face count drift BETWEEN steps — even within one vendor's chain. Check the bounding box and origin after every stage in a DCC or with trimesh; "autoSize"-style meters options live in node transforms that downstream steps may or may not honor.
- **Read input caps before chaining**: one unwrapper caps at 30k faces → set the generator's limit under it, or remesh down first. Schema silence about UVs = prefer a member with explicit keep-existing-UVs.
- **Siblings formats**: when a step outputs GLB beside OBJ/FBX, feed the GLB onward (unless the next step demands otherwise or quads matter — quads travel in FBX only).

## 5 · Repairing the draft (always)

AI meshes arrive with the pathology catalog. The standard intake (formats.md §2 is the toolbox):

1. **Analyze first, always** — never trust, even "it previewed fine": watertight? winding? how many shells (a 5-shell "single" object has 4 islands of junk)? extents vs intent?
2. **Delete the junk shells** (the inner geometry, the floating blobs, the ground plane the model came welded to).
3. **Merge verts → remove degenerate → fix normals** (the order matters — craft.md §4).
4. **Decimate to budget** + volume-delta check (a repair that removes 40% of the volume amputated the subject's arm — look at the render).
5. **Re-texture strategy**: if normals were flipped at generation, the texture is lighting-baked wrong — regenerate texture on the repaired mesh rather than trusting the original maps.
6. **"Delight"** where offered: strip baked-in lighting so the mesh relights in the engine; keep it only when painted shading IS the art style.

## 6 · Multi-view / orbit generation — never guess angles

Asking an image model to "show it from behind" guesses the geometry — **each independently-generated angle drifts 7–53% in scale between views** (scenario-labs' measured finding). The reliable route puts a 3D step in the middle:

```
subject image → image-to-3D (untextured) → clay render per camera →
image model repaints each clay layout from the original → consistent views
```

The mesh fixes silhouette, scale, perspective; the original picture supplies identity and color. Repaint the grounded clay layout (subject standing on the floor WITH its cast shadow, camera fixed) — not a floating cutout over a flat background. Regenerate only the views that fail.

For character turnarounds (reference generation, astra's discipline): 8 horizontal views at 45° spacing + top view, all generated from the SAME original image and feature constraints, orthographic, same scale/camera height — then cross-checked for mutual consistency before use. Generated views are design aids; the original image is the authority.

## 7 · What generation is NOT for

- **Exact dimensions** — a "40 mm bracket with M4 holes" from a generator is a wish; that's cad.md.
- **Editable structure** — the mesh is a fused blob with maybe named parts; a hinge that must open is DCC work.
- **Quad topology promises** — generators output triangles (some offer quad switches → FBX only).
- **Identity/likeness of real people or brands** — fabricating recognizable logos/likenesses is the same no-fabrication rule the video skill enforces; original characters and explicit fan-art policy territory only, with the user owning that call.
