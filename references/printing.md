# 3D printing — from mesh to a part that survives the plate

*Merged from: heyixuan2/bambu-studio-ai (ground rules, analyze-everything, orientation policy, know-the-size), EdwinjJ1/3d-print-skill (manifold-safe-output discipline, surveyed-unlicensed concept), flowful-ai/cad-skill (parametric-printable concept, PolyForm NC — concept only).*

A 3D printer is a physical machine that runs unattended for hours with hot parts. Mistakes waste filament, time, and occasionally hardware. The agent's job ends at analysis, orientation, and preview — **the user presses Print**, because AI-generated meshes have defects analysis can't catch, and the human review is where people catch them.

## 1 · Ground rules

1. **The user starts the print.** The agent never starts, pauses, or changes a print. It opens the model in the slicer; the human reviews and clicks. Don't say a print has started until a status readout says so.
2. **Analyze EVERY model** — downloaded, generated, or user-supplied. No source is trustworthy: `analyze --repair` catches wrong units, floating parts, thin walls, and plate-oversize. Nothing reaches the slicer unaudited.
3. **Show the preview before the slicer.** The user sees the model (render, contact sheet, or slicer preview) before spending time slicing it.
4. **Know the size BEFORE generating.** Generation costs credits and minutes, and scale is the thing most often gotten wrong. If the user didn't give a size, ask: "How big? e.g. 80 mm tall." The same question before long modeling sessions.
5. **Search before generating** — MakerWorld/Printables first (license check per file!), generation second. A tested existing part beats a fresh guess.

## 2 · Printability gates (run on every mesh)

| Gate | Threshold (FDM) | Why |
| --- | --- | --- |
| Manifold/watertight | required | Slicers misheal non-manifold → missing walls |
| Wall thickness | ≥ 1.2 mm structural (≥ 2.4 = 3 perimeters) | Sub-perimeter walls are holes-in-waiting |
| Bed fit | bbox ≤ plate (220×220×250 typical; CHECK the printer) | Split or scale — ask, don't decide |
| Floating geometry | none (unconnected shells) | It prints in mid-air → spaghetti |
| Overhangs | > 45° from vertical needs support or redesign | Gravity doesn't negotiate |
| Holes | ≥ 2 mm diameter vertical, ≥ 3–4 mm horizontal | Smaller closes up (FDM) |
| First layer | real contact patch, not a knife edge | Adhesion is the whole first law |

Resin (SLA) shifts the numbers: tolerances tighter (±0.05 vs ±0.2 mm), walls can be 0.4–0.5 mm, holes fine at 0.5 mm, but drainage and suction matter (hollow with escape holes ≥ 2 mm).

## 3 · Orientation — the free quality decision

Orientation sets strength, surface quality, and support cost, and it's chosen BEFORE slicing:

- **Strength**: layer lines are weak planes — orient so expected load pulls ACROSS layers, not along them. A hook prints vertical; a shelf bracket prints flat on its back.
- **Aesthetics**: the plate-facing surface is the smoothest (and supported scars are the ugliest) — put visible faces up and away from supports.
- **Supports minimized**: chamfer/tear-drop overhangs in CAD instead of printing support towers; tilt flat-topped overhangs past 45°... or split the part.
- **Parametric parts print as designed** (already oriented by construction); downloaded and AI-generated models arrive in arbitrary orientations — analyze and re-orient those, never trust their default pose.

## 4 · Tolerances and fits — the numbers that make parts work

- **Clearance for mating parts**: 0.2–0.3 mm per side (FDM) for sliding fits; 0.1 mm for press; 0.0 + same-slice for fused. Resin: halve them.
- **Holes undersize on FDM** (extrusion pulls inward): drill-sized functional holes get +0.15–0.25 mm on the nominal — an M4 bolt wants a 4.2 mm hole, and `$fn ≥ 48` or it's a polygon under the bolt.
- **Threads**: print-in-place taps at ~50% engagement with 0.2 clearance, or model true threads in CAD for big/loaded ones.
- **Bridging**: < 30 mm spans bridge fine-ish (sag ~0.2 mm); anything else gets support or a redesign.
- **Tolerance is a PARAMETER** in the CAD file (cad.md §4) — per-printer, per-material, regenerated not re-drawn.

## 5 · Slicer handoff

- **Deliver 3MF over STL** when anything matters beyond raw triangles: 3MF carries units (STL's mm are a convention!), part names, colors, multi-object layout. STL is the unitless lowest common denominator — fine for single parts, never for assemblies.
- **Repair before the slicer, not in it**: slicer "auto-repair" is a black box that can amputate; run the trimesh pipeline (formats.md §2) and SEE what repair did (volume delta, shell count) before handing over.
- **AMS/multi-color** (Bambu lane): colors are material assignments per part/facet — 3MF carries them; the AMS maps them to slots.
- Slicer settings are the user's machine territory (profiles live with the printer); the agent's deliverable is a validated, oriented, right-sized mesh — plus the manifest saying so.

## 6 · The print-readiness report

State it in numbers before handoff — the pre-flight the user reads:

```
PRINT-READY: bracket-v3.3mf
  watertight: yes | shells: 1 | faces: 8,412
  bbox: 78.0 × 42.0 × 24.0 mm (plate: 220×220×250 — fits, 4 copies)
  walls: min 2.4 mm verified by section
  orientation: flat on back (load across layers; top face clean)
  overhangs: worst 35° — no supports needed
  holes: 2× M4 @ 4.2 mm, vertical
  printability: PASS all gates (printing.md §2)
```

If any line is a lie, the plate finds out in hour two.
