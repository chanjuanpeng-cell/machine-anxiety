# Machine Anxiety Version Map

This file records the current public and preview versions of the project.

The reusable system structure is documented in [PROJECT_TEMPLATE.md](PROJECT_TEMPLATE.md).

## Stable Submitted Version

- Branch: `main`
- URL: https://machine-anxiety.vercel.app
- Purpose: original submitted / competition-safe version
- Notes: keep unchanged while existing submissions are under review.

## Data Mapping V2

- Branch: `data-mapping-v2`
- URL: https://machine-anxiety-git-data-mapping-v2-chanjuanpeng-cells-projects.vercel.app/
- Purpose: updates anxiety mapping from rate-height-only to deviation, velocity, and shock.

## Sound V2

- Branch: `sound-v2`
- URL: https://machine-anxiety-git-sound-v2-chanjuanpeng-cells-projects.vercel.app
- Purpose: builds on `data-mapping-v2` with layered data-driven sound.

## Visual V2

- Branch: `visual-v2`
- URL: https://machine-anxiety-git-visual-v2-chanjuanpeng-cells-projects.vercel.app
- Purpose: builds on `sound-v2` with subtle directional point-cloud pressure.

## Model Replacement V2

- Branch: `model-replacement-v2`
- URL: https://machine-anxiety-git-model-re-8f1ad7-chanjuanpeng-cells-projects.vercel.app
- Purpose: builds on `presentation-stability-v2` by grouping model, camera, pressure, data, and export settings into a reusable configuration layer.
- Guide: [MODEL_REPLACEMENT_GUIDE.md](MODEL_REPLACEMENT_GUIDE.md)

## Export Pipeline V2

- Branch: `export-pipeline-v2`
- URL: https://machine-anxiety-git-export-p-ea9bad-chanjuanpeng-cells-projects.vercel.app
- Purpose: builds on `model-replacement-v2` with clearer still-image and JPG-sequence export controls.
- Guide: [EXPORT_PIPELINE_GUIDE.md](EXPORT_PIPELINE_GUIDE.md)

## Shader Deformation V2

- Branch: `shader-deformation-v2`
- URL: pending Vercel preview
- Purpose: builds on `export-pipeline-v2` with pressure slices, shear, local fracture, and velocity lag in the point-cloud deformation system.
- Guide: [SHADER_DEFORMATION_GUIDE.md](SHADER_DEFORMATION_GUIDE.md)

## Working Rule

Do not merge preview branches into `main` until the submitted version no longer needs to remain unchanged.

Use `presentation-stability-v2` as the current mother-system candidate for future branches unless a later branch is explicitly promoted.
