# Parametric CAD — exact geometry as code

*Original content for this repo (MIT). The parametric-printable concept is credited to `flowful-ai/cad-skill` (PolyForm Noncommercial — concept acknowledged, content NOT merged, license incompatible). OpenSCAD 2021.01 stable / dev builds; CadQuery 2.8.0 and build123d 0.13.0 live-verified on PyPI 2026-10-15.*

CAD is the correct lane when the model is **defined by dimensions**: brackets, enclosures, spacers, cases, fixtures, adapters. If the user says "40 mm wide, M4 holes, 2 mm walls" — that is CAD, not sculpting. The model is a function of named parameters; change a number, the part regenerates.

## 1 · When CAD vs mesh — the honest split

| Signal | Lane |
| --- | --- |
| Dimensions, holes, threads, tolerances, flat/straight faces | Parametric CAD |
| "It must fit this other part / this bolt / this port" | Parametric CAD |
| Organic form, character, terrain, concept shape | Mesh (blender.md / generative.md) |
| "Looks like this picture" | Mesh, reference-driven |
| Printable functional part | CAD, then printability gates (printing.md) |

Don't sculpt a bracket (no hole is ever round again); don't CAD a dragon (every scale a boolean).

## 2 · OpenSCAD — CSG as a language

OpenSCAD models are **constructive solid geometry**: union (+), difference (−), intersection (∩) of primitives and extrusions. There is no mesh to push around — the part IS the expression, which is why agents are unusually good at it.

```scad
// parametric-box.scad — every dimension is a named parameter
wall  = 2.4;          // mm — wall thickness (3-wall rule for FDM: ≥ 2.4 = 3 perimeters)
inner = [58, 38, 22]; // inner cavity, mm
lid_gap = 0.25;       // tolerance — clearance for the lid to slide (printing.md §4)

module box() {
    difference() {
        // outer shell = inner cavity grown by wall, with rounded corners
        minkowski() {
            cube(inner);
            sphere(r = wall, $fn = 48);
        }
        translate([wall, wall, wall]) cube(inner);
    }
}
module lid() {
    translate([0, 0, wall + inner.z + lid_gap])
        linear_extrude(wall) offset(r = wall) square([inner.x, inner.y]);
}
box(); lid();
```

The disciplines that separate working OpenSCAD from slop:

- **`$fn` is a quality dial, not a default.** `$fn = 12` previews fast; `$fn = 48–96` is the render/export tier. NEVER `$fn = 0`+`$fa`/`$fs` defaults for final parts (too faceted), never `$fn = 360` (pointless — 96 is visually circular at print scale). Set it per-feature: a bearing hole needs more segments than a fillet bead.
- **Preview (F5) is not render (F6).** Preview draws approximations; boolean correctness and manifoldness are only guaranteed after render. Agent form: `openscad -o out.stl model.scad` (render is implicit in export; add `--render` to force full CGAL evaluation when artifacts are suspected).
- **Coincident faces z-fight and mis-boolean**: in a `difference()`, if the cutting cube's face is EXACTLY coplanar with the target's face, the result is undefined flicker. Either overshoot the cutter decisively (`translate([-1,-1,-1])`) or offset by a hair. Same for `intersection()` tangencies.
- **Circles/cylinders are polygons**: an "8 mm hole" at `$fn = 24` is a 24-gon — measurably under-size on bolts. Holes need `$fn ≥ 48` AND a tolerance allowance (printing.md).
- **Modules = named functions**: `module m4_hole()` used 6 times means one place to fix the tolerance. Copy-pasted cylinder triples are the CAD version of magic numbers.
- **`linear_extrude`/`rotate_extrude` with `scale`/`twist`** for tapered and revolved forms; `text()` + `linear_extrude` for embossed labels (the workhorse of nameplates — recipes R03); `offset()` in 2D before extrude for padding/shrink — the tolerance tool.
- **minkowski() is powerful and SLOW** — rounded-box shells in one line, but render time scales badly; for production parts, `hull()` of two offset shapes is often 10× faster to the same visual.

CLI pipeline (the agent form):

```bash
openscad -o bracket.stl --export-format binstl bracket.scad     # render + export
openscad -o preview.png --imgsize 1200,900 --projection p bracket.scad  # visual check
# then the numbers gate (craft.md §6) on the STL before delivery
```

## 3 · CadQuery / build123d — Python parametric

When the part needs fillets, chamfers, workplanes on faces, or logic loops — Python beats CSG text. CadQuery (2.8.0) is the fluent API; build123d (0.13.0) is the newer algebraic core — same author lineage, similar power, pick per project taste.

```python
import cadquery as cq

# parametric L-bracket: sketch → extrude → features
t, h, w = 3.0, 40.0, 60.0        # mm — thickness, arm height, arm width
hole_d, hole_pos = 4.2, 8.0      # M4 bolt: 4.2 = 4.0 + 0.2 print tolerance

bracket = (
    cq.Workplane("XY").box(w, h, t)                     # base plate
    .faces(">Z").workplane()
    .box(w, h, t)                                       # vertical arm (relocated)
    .edges("|Z").fillet(3.0)                            # soften vertical edges
    .faces(">Z").workplane()
    .pushPoints([(-hole_pos, -hole_pos), (w - hole_pos, -hole_pos)])
    .hole(hole_d)                                       # through-holes
)
cq.exporters.export(bracket, "bracket.stl")
```

- **The grammar**: `Workplane` → 2D sketch (`rect`, `circle`, `polyline`, `slot2D`) → `extrude`/`revolve`/`loft` → face/edge selectors (`>Z`, `|X`) → features (`fillet`, `chamfer`, `hole`, `shell`, `cut`). Selectors are the superpower: "fillet the edges parallel to Z" without naming any edge.
- **`shell()`** hollows for enclosures (wall thickness in one call). **`cut()`/`union()`** compose parts; **`LOFT`** bridges sketches — the enclosure-with-complex-top lane.
- **Exporters**: STL for print, STEP for CAD interchange (STEP carries units and B-rep — the format engineers expect), GLB via conversion (Blender lane) for web.
- **build123d differences worth knowing**: explicit `BuildPart`/`BuildSketch` contexts, list-based boolean ops, better type checking. Either way: same parametric discipline — named constants at top, one function per feature, comment every measured constant with its source.

## 4 · The parametric discipline (applies to both)

1. **Every dimension is a named parameter with a comment**: `bolt_clearance = 0.2; // FDM fit for M4, measured 2026-10` — never a bare literal deep in geometry.
2. **One module/function per feature**; assembly = module composition. The part tree must read like a parts list.
3. **Tolerances are parameters too** (fit, clearance, shrink) — they vary per printer/material; the model regenerates, it doesn't get re-drawn.
4. **Regenerate, never hand-patch**: a "quick manual fix" in a CAD export is a fork of the truth — the next parameter change silently drops it.
5. **The output is still a mesh**: STL from OpenSCAD/CadQuery goes through the SAME numbers gates (watertight, winding, extents) as everything else — CAD-generated does not mean CAD-validated. Run craft.md §6 on every export.
6. **STEP vs STL**: deliver STEP when a human will re-open it in CAD; STL/3MF when it goes to a slicer; never .blend for a bracket.

## 5 · Common CAD bugs (the ones agents actually hit)

- **Under-size holes**: polygon approximation + no tolerance. Fix: `$fn ≥ 48` AND `+0.2 mm` (FDM) / `+0.1` (resin) on functional holes.
- **Boolean flicker**: exact-coplanar faces in difference/intersection. Fix: overshoot cutters past the surface.
- **Zero-thickness walls**: a difference that exactly grazes an outer face leaves a knife-edge. Fix: make cutters bite decisively.
- **Unprintable float**: geometry hanging in air with no path to the plate (CAD doesn't warn — printing.md §3 catches it).
- **minkowski/hull explosion**: render hangs; almost always `$fn` too high on a minkowski'd sphere. Fix: lower the rounding resolution, or hull() instead.
- **The invisible unit**: OpenSCAD is unitless-by-convention-mm; if anything upstream said meters, the bracket is 1000× off. The extents gate catches it — always run it.
