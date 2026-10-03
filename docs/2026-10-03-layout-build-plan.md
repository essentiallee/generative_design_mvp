# Office layout MVP: source assessment and first build

Date: 2026-10-03
Status: implementation plan; no Rhino/GH model or solver has been executed yet.

## 1. Source assessment

- Source: user-supplied `평면 레이아웃 룰.pdf`, one wide page.
- Left: base building shell with an open workplace floor, structural columns, and an upper fixed core (toilets, stair, lift/landing and corridor/lobby).
- Right: two raster fit-out references annotated with program types and circulation paths. These are references, not CAD geometry or ground-truth room dimensions.
- PDF contains 2,784 line objects, 7,069 curve objects, 206 rectangles and two raster images. The images occupy the right-hand reference regions; the base has vector linework worth testing through direct PDF import.
- Horizontal grid A-D: 8,400 + 8,300 + 9,200 = 25,900 mm.
- Vertical grid 1-6: 5,350 + 1,500 + 5,500 + 6,100 + 4,600 = 23,050 mm.
- These are grid dimensions, NOT the usable office dimensions. Grid 1 is visibly below the lower enclosure; do not create a 25,900 x 23,050 rectangular office from these labels.
- The dense fine grid in the open floor is reference/finish linework, not room partitions.
- A double-door symbol is visible near the upper-right interface to the open workplace. Confirm its role, width, and swing before using it as the tenant entrance.
- Notes propose four programs: Type 01 Office; Type 02 Operation Room; Type 03 Work Room (desks/chairs); Type 04 Prep Room. Public/open area is separate. Program labels need user-confirmed sizes, counts and adjacency requirements; do not infer clinical or occupancy rules from names.
- Unresolved: usable boundary, wall thicknesses, exact column footprints, entrance designation, doors and swings, furniture footprints, room sizes and requested counts.
- Document notes proposing inches, automated full layouts and a 1,500 mm Korean-standard hallway are proposals/reference annotations, not adopted project requirements.
- Keep canonical units in mm. 1,500 mm is a provisional user/reference width, not a verified legal minimum. Unit conversion alone does not establish US compliance.

## 2. Immediate milestone: calibrated semantic base

Deliverables to create in Rhino next:
- `models/rhino/office_base_v001.3dm`
- `models/grasshopper/office_base_validation_v001.gh`
- `data/model_manifest.json` with units, tolerances, object IDs, roles, source and confidence
- `results/base_validation_v001.json` and a labeled top-view screenshot

Steps:
1. Open a millimeter Rhino model; set a documented tolerance (0.1 mm is a proposed prototype setting, not source accuracy).
2. Import the existing vector PDF into a reference layer. Do not make another PDF first. If an original DWG/DXF exists, prefer that as the geometric source.
3. Isolate the left base; lock the right reference pictures and annotations separately. Never use the page border as the office boundary.
4. Calibrate with A-D = 25,900 mm; independently check A-B = 8,400 mm and a vertical span such as 5-6 = 4,600 mm. Use measured scale, not the printed 1:75 label on the composed page.
5. If horizontal and vertical scales disagree materially, investigate distortion rather than stretching the model to fit. Record actual calibration errors and chosen acceptance threshold.
6. Draw/simplify the usable tenant perimeter at interior finished faces, excluding the core. Trace column footprints and other fixed obstructions. Do not offset unknown wall thicknesses to guess the usable area.
7. Use local XY with Z=0; preserve a record of translation/rotation/scale applied to the source.
8. Identify entrance thresholds and keep-clear areas; mark unknown dimensions explicitly.
9. Save the base before adding any candidate partitions or hallway alternatives.

Suggested layers:
- `00_REF_PDF` (locked)
- `01_USABLE_BOUNDARY`
- `02_FIXED_CORE`
- `03_COLUMNS`
- `04_ENTRANCES`
- `05_EXISTING_WALLS`
- `10_ROUTE_CENTERLINES`
- `11_CORRIDOR_CLEAR_ZONE` (generated)
- `20_PROGRAM_ZONES`
- `21_PARTITION_CENTERLINES`
- `22_DOORS`
- `23_FURNITURE`
- `90_PREVIEW` (generated)
- `91_VIOLATIONS` (generated)

Base acceptance checks:
- Verified scale in both directions; mm recorded.
- Usable perimeter closed, planar, simple and nonzero-area.
- Closed obstacle footprints with stable IDs; no annotation/hatch/grid geometry treated as obstacles.
- No duplicate or zero-length model curves; no silently repaired open boundaries.
- Entrance points/segments linked to the usable floor; uncertain roles remain flagged.
- Net area computed from the actual usable region minus the union of internal obstacles, without subtracting the same core twice.

## 3. Columbia tutorial adaptation

5.4: Keep manually drawn inputs, named parameters, centerline offsets, region operations and explicit outputs. Map streets to circulation, blocks to available program zones. Do not copy automatic parcelization as an office-room solver. Do not extend a corridor blindly through the core or facade.

5.5: Keep a library of program types and rule-based selection. Place dimensioned modules by translation/rotation; use permitted size ranges for variable rooms. Do not Box Morph doors, furniture or required clearances. Defer attractor/density logic until a specific measurable design goal requires it.

First hallway test:
- Draw one route from the verified entrance through a small selected test zone.
- Define width as TOTAL CLEAR WIDTH. If 1,500 mm is adopted for the experiment, offset the centerline by +750 and -750 mm.
- Cap ends; union branches; inspect inside corners, junctions and self-intersections.
- Generate a raw corridor envelope, then test containment/collisions. Do not clip away invalid portions and report the remainder as a valid full-width corridor.
- Place walls outside the clear envelope; wall thickness must not consume the required clear width.
- Test route continuity to intended doors, not just total corridor area. Door swing/access zones are separately modeled.

## 4. Introduce Python 3 immediately after calibration

Rhino 8: Maths > Script > Python 3. Use CPython 3, not the legacy IronPython component.

First component: `ValidateBase`
Inputs: Boundary (Curve, Item), Obstacles (Curve, List), Entrances (Point/Curve with explicit convention, List), ToleranceMm (Number, Item).
Outputs: IsValid, NetAreaM2, Issues, InvalidGeometry.
Rules: boundary closure/planarity/simplicity; valid obstacles; meaningful entrance placement; unit/tolerance checks; area by region operations, not naive overlapping sums.

Second component: `BuildCorridor`
Inputs: validated model, route centerlines, ClearWidthMm, Run.
Outputs: raw clear envelope, violations, feasible region, connectivity status.

Third component: `TestPartitionMove`
Inputs: selected partition ID, movement direction and range, linked doors, required clear zone, fixed objects and explicitly movable furniture.
Outputs: checked candidates, measurements, blocker IDs, computation status.

Use small component wrappers importing shared Python from repository `src/office_layout/`. Configure the repository src path in Rhino's Python search paths or a local # env directive; do not commit machine-specific absolute paths. Ordinary sliders update local geometry; paid AI requests require an explicit action.

Model state separates:
- Geometry: referenced curves and footprints.
- Roles: room/partition/door/column and stable source IDs.
- Parameters: permitted movements and bounds.
- Hard requirements: locked objects, width, containment and collision checks.
- Goals: target area, fewer moved desks, shorter movement.
- Results: exact tested parameter values, valid/invalid/unknown, blocker IDs and metrics.

GH calculations produce previews, not baked duplicates. Apply/bake is explicit and undoable. Run pure computations on geometry copies; document and canvas mutations must occur on the proper Rhino/GH thread. Detect changed source state before applying an older preview.

## 5. First experiment before full layout generation

- Use one small portion of the real base; add one room partition and explicitly dimensioned test desks if needed. Record those as prototype fixtures, not source-plan furniture.
- A: vary partition offset while all desks stay fixed.
- B: allow selected unlocked desks to relocate among bounded candidate positions.
- Check every complete candidate against boundary, columns, corridor, door and furniture conditions.
- Start with 50 mm search steps, then refine around feasible limits; state resolution.
- Report largest tested feasible movement, not a universal maximum. Do not assume feasibility is monotonic for arbitrary geometry.
- The earlier 300/600 mm example is illustrative. Never hard-code it as this plan's result.
- Only say 'requires moving two desks' if the tested search establishes that minimum within its stated domain; otherwise say 'found an option moving two desks'.
- Regression cases: exact threshold, 1 mm violation, column collision, blocked door, disconnected corridor, locked desk, no feasible option and wrong units.

## 6. Language and MCP: after deterministic checks work

Stage A: no AI or local server. GH calls local Python directly.
Stage B: structured language interpretation. User selects geometry and asks for an allowed edit. A model proposes object IDs, operation, bounds and goals. Validate the request against a schema and known IDs; execute the existing Python engine. The model does not invent measured results.
Stage C: optional MCP adapter for an external assistant. Use a separate Python process with the official MCP Python SDK; local stdio is a suitable first transport. MCP is a tool interface, not a geometry solver or a prerequisite for this plugin.

Proposed tools:
- inspect_model()
- check_constraints(model_version)
- preview_partition_change(partition_id, bounds, goals, model_version)
- apply_candidate(candidate_id, model_version) only after user selection

For a desktop bridge, the adapter queues typed requests for Rhino/GH to execute on its own thread and returns request IDs/results. Do not call Rhino document mutations from a network worker. Do not expose arbitrary exec/eval. Keep secrets outside the repository; send only task-relevant model context. Add timeouts, cancellation and stale-result checks. If the panel calls an LLM directly, MCP can remain optional.

Later distribution: Rhino Script Editor can publish Python commands/components and shared Python libraries as Rhino/GH plugins. Verify on the target Rhino version before distribution; no planned C# rewrite is required.

## 7. Recording and repository workflow

Confirmed local checkout: `/Users/jaehyunlee/Documents/GitHub/generative_design_mvp`
Confirmed remote: `https://github.com/essentiallee/generative_design_mvp.git`
Observed initial state: clean main branch, starter Readme.md and hello.txt only.

Suggested structure (future files, not yet implemented):
```
docs/                 source assessment, setup, decisions and experiment log
models/rhino/         milestone .3dm files
models/grasshopper/   paired .gh files; optional .ghx exports for inspection
src/office_layout/    shared Python source
components/           small Python component wrappers
config/               units, rules and confirmed typology definitions
data/                 stable-ID manifests and source provenance
results/              measurements, candidate records and preview images
tests/                deterministic geometry and rules tests
```

- Keep original architectural references unchanged locally; source PDF is not included in the public repository by this planning step.
- Save paired .3dm/.gh milestones and the matching Python/config changes together.
- Record Rhino version, component dependencies, units, tolerance, source calibration, assumptions, test inputs and outcome.
- Commit readable .py/.json/.md alongside binary CAD files; .ghx helps inspection but is not guaranteed to merge cleanly.
- Use Git LFS when binaries warrant it; agree tracking before large uploads.
- Exclude credentials, .env, caches, autosaves and private project materials.
- A local Git checkout is not an HTTP server. No web server is needed for the first GH prototype.

## References
- Columbia 5.4: https://smorgasbord.cdp.arch.columbia.edu/modules/5-computational-design-modeling-in-grasshopper/5-4-street-grid/
- Columbia 5.5: https://smorgasbord.cdp.arch.columbia.edu/modules/5-computational-design-modeling-in-grasshopper/5-5_buildings-density/
- Rhino PDF import: https://docs.mcneel.com/rhino/8/help/en-us/fileio/portable_document_format_pdf_import_export.htm
- Python 3 components and module paths: https://developer.rhino3d.com/guides/scripting/scripting-gh-python/
- Python plugin packaging: https://developer.rhino3d.com/guides/scripting/projects-create/
- Rhino threading: https://developer.rhino3d.com/guides/scripting/advanced-async/
- MCP server guide: https://modelcontextprotocol.io/docs/develop/build-server
- SmartPlan positioning reference: https://www.smartplanai.com/solutions/for-brokers
