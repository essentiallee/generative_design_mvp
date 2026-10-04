# Office layout prototype — 2026-10-04 checkpoint

## Downloads

Download both files in the latest working pair:

- [Rhino model: v02 clean](https://github.com/essentiallee/generative_design_mvp/raw/refs/heads/main/model/261003_mvp_base_v02_clean.3dm)
- [Grasshopper definition: v02 fx](https://github.com/essentiallee/generative_design_mvp/raw/refs/heads/main/model/261003_mvp_base_v02_fx.gh)

Earlier reference:

- [Rhino model: v01](https://github.com/essentiallee/generative_design_mvp/raw/refs/heads/main/model/261003_mvp_base_v01.3dm)

## Open

1. Open the v02 clean model in Rhino 8. Model dimensions use millimeters.
2. Open the v02 fx definition in Grasshopper. It uses Python 3 components.
3. If a referenced geometry input is missing, reselect its corresponding geometry in the Rhino model.

## Recorded checkpoint

- Adjustable circulation width and remaining office-area calculation.
- Desk footprint and desk-plus-chair-zone containment checks.
- A sampled +Y movement search with a preview and verification of the proposed chair zone.
- In the demonstrated test: desk PASS, original chair zone FAIL, proposed chair zone PASS after a 400 mm move, using 50 mm search increments.

The movement is the first passing sample along +Y, not a global minimum. Chair clearance is a prototype parameter. Furniture-to-furniture collisions and regulatory compliance are not evaluated.

These native files were saved and tested by the author in Rhino/Grasshopper; they have not been independently reopened as part of this upload.
