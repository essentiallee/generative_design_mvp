# Office layout generator — contributor and AI handoff

Checkpoint: 2026-11-09. This describes the latest working Rhino/Grasshopper prototype, including the batch row-offset experiment. Read this file before modifying the definition. The workflow below is documentation, not a guarantee that every saved UI label exactly matches it; verify component code and input settings in the `.gh` file.

## 1. Files and environment

The only current native model pair is:

- [261003_mvp_base_v02_clean.3dm](https://github.com/essentiallee/generative_design_mvp/raw/refs/heads/main/model/261003_mvp_base_v02_clean.3dm) — updated architectural base and referenced geometry. Its filename date does not indicate the latest save date.
- [261109_mvp_base_v03_fx.gh](https://github.com/essentiallee/generative_design_mvp/raw/refs/heads/main/model/261109_mvp_base_v03_fx.gh) — latest Grasshopper definition, Python scripts, batch generation, previews, and graphs.

Open the `.3dm` first, then the `.gh`, using Rhino 8 for macOS and Grasshopper Python 3 (CPython), not legacy IronPython. Canonical units are millimeters; plan geometry lies in World XY, Z=0. Use the Rhino document tolerance in geometric checks. The implementation uses native GH components, RhinoCommon, and Python standard libraries. This workflow does not require MCP, an LLM, a local web server, or a paid optimization plugin. Compatibility with other Rhino builds has not been independently verified.

Older native files have been removed from the current folder; their committed revisions remain in Git history. Do not recreate obsolete examples as active dependencies.

## 2. Goal and present scope

Explore office arrangements by varying cubicle dimensions and row positions, retaining compatible placements, and comparing measurable outcomes. The current system is a deterministic, seeded batch design explorer. It is not yet an adaptive genetic optimizer, a pedestrian simulation, or a complete office-program solver.

Current experiment variables: desk width/depth, chair clearance, partition thickness, and row-offset fraction. Fixed study inputs include the architectural shell, columns, circulation geometry, side gap, and inter-cubicle gap. Occupancy is a target threshold; batch count is deliberately not capped at that threshold.

The earlier architectural program names were Office, Operation Room, Work Room, and Prep Room, plus public space. The current cubicle batch does not allocate or validate those other programs. Do not interpret its count as capacity after meeting the full program brief.

## 3. Architectural inputs and geometry conventions

- The base was calibrated in mm, including A–B = 8400 mm. Use the traced usable boundary, not the PDF page or a bounding rectangle.
- The usable boundary intentionally has notches around columns. Column outlines are separate reference geometry. Avoid subtracting excluded geometry twice.
- The entrance threshold, closed public-area outline, and route centerline are distinct inputs.
- Circulation was constructed from positive/negative half-width route offsets, combined with the public area while preserving the central opening.
- Office-area surfaces represent the usable region outside circulation, including the central island and peripheral regions. Preserve their holes and notches.
- Corridor width is total clear width. Trial dimensions and clearances are not verified regulatory requirements.
- GH outputs are previews; do not automatically bake duplicates or overwrite referenced Rhino geometry.

## 4. Active pipeline

```text
Rhino references → circulation and office-region construction
Circulation Brep → Deconstruct Brep edges → Explode Curve segments
                                                  ↓
                                        Candidate Frontages
                                                  ↓
Frontages + Office Breps + dimension-option lists → Batch Experiments
                                                  ├→ JSON experiment records
                                                  ├→ trade-off points
                                                  └→ best-count progress points
JSON records + zero-based index → Preview Experiments → cubicle geometry
                                        └→ independent fit/collision checks
JSON records + same index + graph origin → Plot Experiments
```

There may also be standalone `Generate Candidate Rows` and `Select Compatible Cubicles` components from the intermediate implementation. The batch script implements those operations internally. Modifying standalone copies does not automatically update the batch; inspect which components actually feed the active preview.

## 5. Cubicle module

The center input is the desk center, not the cubicle-envelope center. Local −Y is the open front; left, right, and rear have walls. Chair clearance extends behind the desk toward the open front. Wall thickness grows outside the clear interior.

```text
clear interior width = width + 2 × side_gap
clear interior depth = depth + clearance
overall width = width + 2 × side_gap + 2 × wall_t
overall depth = depth + clearance + wall_t
```

Each cubicle owns three walls. Shared-wall construction has not been implemented. All geometry rotates together around the desk center: desk, chair zone, walls, envelope, and access point. Positive angles are counterclockwise in Top view: 0° opens −Y, 90° +X, 180° +Y, 270° −X.

Rotation-enabled module inputs are `center` (Point3d/Item) and `width`, `depth`, `clearance`, `side_gap`, `wall_t`, `angle` (float/Item). Regular outputs are `desk_curve` (desk), `b` (chair zone), `c` (three wall curves), `d` (envelope), and `e` (front midpoint).

**Python socket rule:** preserve the special `out` socket for printed messages. It is normally empty. It is not a geometry output. Input variable names and regular output names must match script identifiers exactly; changing only a visible nickname may not fix a binding problem. Earlier bugs included `chairzone`/`desk` mismatches and accidentally renaming `out` as a geometry socket.

## 6. Candidate generation and selection logic

1. Extract straight circulation boundary segments, including outer and inner perimeter segments.
2. For each segment, test both sides. Derive a local plane so the cubicle opens toward the frontage.
3. Place slots with pitch `overall_width + max(gap, margin)`, where `margin = max(1 mm, 2 × document tolerance)`.
4. Distribute unused row length with offset fraction 0 (segment start), 0.5 (center), or 1 (segment end). Curve direction defines start/end. One fraction applies to all rows in a batch experiment; offsets are not independent per row.
5. Keep envelopes wholly contained in an office face and disjoint from its holes. Boundary touching is rejected. Access points are offset slightly into the office side; that alone does not prove access.
6. Build a conflict graph among candidate envelopes. Overlap/touch is a conflict. For otherwise disjoint polygons, a separation below `gap - tolerance` is a conflict. **Do not restore `distance <= gap + tolerance`: it incorrectly rejected rows spaced at exactly the requested gap.**
7. Try seeded selection orders, alternating a fewest-conflicts heuristic and shuffled order; keep the largest compatible set found. This is a heuristic result, not a proven maximum.

## 7. Batch Experiments component contract

| Inputs | Type hint / access |
|---|---|
| `frontages` | Curve / List / Flatten |
| `offices` | Brep / List / Flatten |
| `widths`, `depths`, `clearances`, `wall_ts`, `offsets` | float / List / Flatten |
| `side_gap`, `gap` | float / Item |
| `target`, `trials`, `seed` | int / Item |
| `run` | bool / Item |

Option Panels must have Multiline Data enabled: one numeric value per item. The batch enumerates the Cartesian product of option lists. Two choices for each of four dimensions and three offsets yield 48 experiments. Options are sorted and deduplicated; offsets vary fastest in the current enumeration.

Prototype bounds: at most 243 combinations, at most 150 candidate slots per combination, and 1–100 selection trials. Begin with 10 trials. Large studies run synchronously and can occupy GH while calculating. `run=False` clears outputs; it does not pause or preserve results. The same inputs and seed reproduce the same study within the same geometry/software context.

| Regular output | Meaning |
|---|---|
| `a` | List of JSON result strings |
| `b` | Raw trade-off points: X = clear-zone m²/cubicle, Y = selected count |
| `c` | Raw progress points: X = experiment ID, Y = running best count |
| `d` | Batch summary/error text |

The batch continues selecting beyond `target`; it measures found geometric capacity and reports target shortfall. It does not output just the requested number.

Record fields include `id`, `width`, `depth`, `clearance`, `wall_t`, `side_gap`, `gap`, `target`, `count`, `shortfall`, `clear_area_m2`, `candidate_count`, `seed`, `trials`, `row_offset`, `access_checked`, `centers`, and `angles`. Centers and angles correspond in list order. Preserve that pairing. `clear_area_m2 = width × (depth + clearance) / 1,000,000`; it excludes side gaps, walls, and shared circulation. It is not a complete comfort metric.

## 8. Preview and graphs

`Preview Experiments` receives `records` (str/List/Flatten) and `index` (int/Item). Index is zero-based; candidate IDs start at 1. For 48 records, valid indices are 0–47. Outputs:

| Output | Meaning |
|---|---|
| `a`, `b`, `c` | Desk curves, chair-zone curves, wall curves |
| `d`, `e`, `f` | Envelopes, access points, selected-result report |
| `g`, `h` | Desk centers and matching rotation angles |

Preview reconstructs geometry from the stored experiment dimensions. Do not feed those centers into a module still using unrelated live dimension sliders. Wall curves feed Boundary Surfaces with its input grafted. Preview envelopes feed collision/fit checks directly, not through wall surfaces.

`Plot Experiments` receives the same records/index and `origin` (Point3d/Item). Outputs `a` all plotted points, `b` selected point, `c` axes, `d` label locations, `e` label strings, `f` selected report. Label locations/text feed Text Tag 3D. Display coordinates are scaled/translated; axis labels express the original metrics. Multiple IDs share a point when their two metrics coincide. The same index must control both preview and plot.

Progress is the running best through an enumerated batch, not evidence of adaptive learning or global convergence. A flat line can mean the first experiment already found the best count. Display-only Scale/Move components may relocate the raw progress points; use a vector for Move T. Changing a downstream preview index should not recompute the upstream batch. Playback of saved results is different from live previews during evaluation.

## 9. Verification and limitations

Before accepting a change:

1. Confirm source references and mm/World-XY assumptions; validate closed office/obstacle geometry.
2. Check one cubicle at 0°, 90°, 180°, 270°; all outputs must rotate together.
3. Test containment with a full fit and with an envelope crossing a boundary.
4. Test collisions using deliberately overlapping cubicles; empty conflict outputs are normal when the summary says PASS.
5. Preview multiple batch indices; geometry, dimensions, reports, and highlighted graph point must agree.
6. Check preview envelopes with the independent boundary and pairwise collision components.
7. Save, close, and reopen the paired native files. Recheck Python bindings and geometry references before committing.

The author reported successful generation, graph linkage, preview switching, and boundary/collision checks. This handoff/upload did not independently reopen the native definition. Numeric counts in past screenshots depended on trial dimensions and should not be used as fixed regression expectations.

Not implemented/verified: usable width-aware access paths to every cubicle and entrance; full office program allocation; door swings and egress validation; code compliance; acoustic/daylight/comfort performance; Pareto-front highlighting; independent per-row offsets; adaptive optimization; automatic application/baking. Columns already excluded by office boundaries are respected through containment; do not claim a separate complete obstacle/access audit.

## 10. Continue from here

The proposed next task is **Pareto-front highlighting** on the trade-off graph: maximize selected count and clear-zone area per cubicle, retaining all designs not strictly dominated by another tested design. Distinguish meeting a fixed occupancy target from maximizing count. Before recommending a usable layout, implement and validate access and restore the other program requirements.

Keep the working batch as a baseline. Add variables incrementally, document bounds and metrics, preserve reproducible result records, and verify against existing geometry checks. Record changes in readable source/docs alongside the `.gh` where practical. The active scripts are embedded in this definition; do not assume an external Python package or server is already connected.

For historical context, see [the earlier build plan](../docs/2026-10-03-layout-build-plan.md). It describes earlier implementation intentions; this checkpoint takes precedence for the current implemented workflow.
