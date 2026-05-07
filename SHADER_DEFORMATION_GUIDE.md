# Shader Deformation Guide

This branch develops a more complex deformation layer for Machine Anxiety while keeping the existing model, data, sound, and export systems intact.

## Current Branch

- Branch: `shader-deformation-v2`
- Based on: `export-pipeline-v2`
- Purpose: add a stronger data-pressure deformation layer that can later become a GPU shader system.

## What Changed

The project still uses CPU point-cloud deformation, but the logic now behaves more like a shader deformation field.

New deformation behaviours:

- pressure-driven scan slices
- horizontal shear
- local fracture offsets
- velocity-based lag
- shock-sensitive point separation

These sit on top of the previous organic point-cloud drift.

## Main Configuration

The deformation settings are grouped in `SYSTEM_CONFIG.deformation` in `index.html`:

```js
deformation: {
    sliceStrength: 0.055,
    sliceFrequency: 6.4,
    shearStrength: 0.045,
    fractureStrength: 0.095,
    lagStrength: 0.055,
    gateThreshold: 0.34
}
```

## Artistic Intention

The deformation should feel like external computational pressure entering the room, not like a decorative glitch.

The intended visual language is:

```text
stable domestic scan
-> data pressure accumulates
-> scan slices begin to slip
-> room edges shear
-> high-volatility moments fracture locally
-> the model partially remembers its original shape
```

## What To Check

- The opening room should still be readable.
- Deformation should become visible gradually.
- The point cloud should not become too white or flat.
- Strong moments should feel spatial, not like a screen filter.
- Export stills should still be usable.
- Browser performance should remain smooth.

## Future Improvements

- Move deformation into a real GPU shader.
- Add named deformation presets.
- Add pressure sources for doors, windows, screens, and body zones.
- Add a debug view for pressure origin and slice planes.
- Add model-specific deformation settings.
