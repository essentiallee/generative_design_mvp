# Office layout prototype — 2026-11-09 checkpoint

## Latest working pair

Download both files:

- [Rhino model: v02 clean (updated)](https://github.com/essentiallee/generative_design_mvp/raw/refs/heads/main/model/261003_mvp_base_v02_clean.3dm)
- [Grasshopper definition: v03 fx](https://github.com/essentiallee/generative_design_mvp/raw/refs/heads/main/model/261109_mvp_base_v03_fx.gh)

Earlier references:

- [Grasshopper definition: v02 fx](https://github.com/essentiallee/generative_design_mvp/raw/refs/heads/main/model/261003_mvp_base_v02_fx.gh)
- [Rhino model: v01](https://github.com/essentiallee/generative_design_mvp/raw/refs/heads/main/model/261003_mvp_base_v01.3dm)

## Open

1. Open the updated v02 clean model in Rhino 8. Dimensions use millimeters.
2. Open the v03 fx definition in Grasshopper. It uses Python 3 components.
3. If geometry references are missing, reselect the corresponding geometry in the Rhino model.
4. Review dimension-option lists and controls before running the batch. The demonstrated study tests 16 dimension combinations at three row offsets, for 48 experiments.
5. Use the preview index to inspect a result and its matching graph point.

## Recorded checkpoint

- Cubicle geometry with editable desk dimensions, chair clearance, side gap, wall thickness, and rotation.
- Automatic aligned row candidates derived from circulation edges.
- Compatible-set selection with boundary and envelope collision checks.
- Batch evaluation of dimension combinations and row offset fractions (0, 0.5, 1).
- Recorded results, experiment preview, trade-off graph, and best-count progress plot.
- User reported successful operation and boundary/collision verification in Rhino/Grasshopper.

## Interpretation

Counts describe geometric fit under the tested rules, not verified usable office capacity or a proven maximum. Circulation access paths, other program allocations, and regulatory compliance are not checked by this batch experiment. Clear-zone area is a space-provision measure, not a complete comfort metric. The progress graph tracks the running best result through a finite batch; the search is not yet adaptive.

These native files were saved and tested by the author; they have not been independently reopened as part of this upload. Earlier revisions of the updated Rhino file remain available in Git history.
