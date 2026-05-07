# Machine Anxiety

A data-driven moving image work exploring how economic systems affect private space.

## Overview

Machine Anxiety is based on a 3D scan of a private room, reconstructed as a point-cloud environment.

The work uses real GBP/CNY exchange-rate data (September 2024 – April 2026) as a continuous input to drive spatial deformation, rotation, and camera movement.

The project exists as a real-time WebGL system driven by historical GBP/CNY exchange-rate data, where past financial fluctuations are reactivated as a continuous computational force.

## Live Version

👉 [View the real-time version](https://machine-anxiety.vercel.app/)

This is the stable submitted version. Experimental preview links are tracked in [VERSIONS.md](VERSIONS.md).

The reusable system structure and future expansion plan are documented in [PROJECT_TEMPLATE.md](PROJECT_TEMPLATE.md).

The first future-work branch for replacing the point-cloud model is documented in [MODEL_REPLACEMENT_GUIDE.md](MODEL_REPLACEMENT_GUIDE.md).

The export workflow for high-resolution stills and JPG sequences is documented in [EXPORT_PIPELINE_GUIDE.md](EXPORT_PIPELINE_GUIDE.md).

The more complex deformation branch is documented in [SHADER_DEFORMATION_GUIDE.md](SHADER_DEFORMATION_GUIDE.md).

## Video

👉 [Watch the final video](https://vimeo.com/1188890187)

## Technical Notes

* Built with Three.js and WebGL  

* Point cloud reconstructed from LiDAR scan  

* ~300k vertices  

* Data-driven system using GBP/CNY exchange rates (JSON)  

* Current preview branches test refined data mapping, layered sound, subtle point-cloud pressure, and loading safeguards.

* Parameters mapped to:

  * spatial deformation

  * rotation speed

  * camera movement

  * visual density

* Offline rendering via CCapture.js (JPG sequence)

* Final video composed in Adobe Premiere Pro

## Running Locally

For the most reliable local preview, run a local server from the project folder:

```bash
python3 -m http.server 5173 --bind 127.0.0.1
```

Then open:

```text
http://127.0.0.1:5173/
```

Opening `index.html` directly may work in some browsers, but a local server is safer for loading the point-cloud and JSON assets. For best performance, use Chrome or Safari.

## Version Management

The `main` branch is kept as the stable submitted version while existing submissions are under review.

Preview branches should remain separate until a new version is ready to replace the public production URL. See [VERSIONS.md](VERSIONS.md) for the current branch and deployment map.

For future works derived from this project, use [PROJECT_TEMPLATE.md](PROJECT_TEMPLATE.md) as the system checklist before changing models, data sources, visual deformation, sound, export settings, or external software connections.

For new scans or model-based works, start with [MODEL_REPLACEMENT_GUIDE.md](MODEL_REPLACEMENT_GUIDE.md).

For still images and video-sequence output, use [EXPORT_PIPELINE_GUIDE.md](EXPORT_PIPELINE_GUIDE.md).

For data-pressure slice, shear, and fracture experiments, use [SHADER_DEFORMATION_GUIDE.md](SHADER_DEFORMATION_GUIDE.md).

## Data
The dataset consists of historical GBP/CNY exchange-rate values recorded between September 2024 and April 2026.
Data is used as a continuous input to drive the system, rather than being visualised directly.

## License
Artwork and code © Peng Chanjuan, 2026
All rights reserved.
This repository is shared for viewing and reference only.
Please do not reuse or redistribute without permission.
