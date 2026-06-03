# Machine Anxiety — Architecture

> A single-file, vibe-coded WebGL artwork. All runtime logic lives in one ES module `<script>` inside `index.html` (~1865 lines).
> This document traces the main thread — *how exchange-rate data becomes image and sound* — with line numbers for navigation.

---

## In one sentence

A real-time system where **exchange rate → an `anxiety` scalar → drives point-cloud deformation, camera movement, post-processing glitch, and generative audio** all at once.

```
chronicle_data.json (25 GBP/CNY rate points)
        │
        ▼
getMappedAnxiety()        ── converts each rate into stress signals (deviation / velocity / shock)
        │                    and combines them into the master scalar `anxiety`
        ▼
getDeterministicErosionMemory()  ── accumulates an irreversible "damage memory"
        │
        ▼
animate() drives each frame ──┬── per-vertex point-cloud deformation (incl. ceiling/wall erosion)
                              ├── camera (cinematic shots / auto-orbit / manual)
                              ├── post-processing (Bloom / RGB glitch / fog)
                              └── Web Audio generative soundscape
```

---

## Tech foundation

- **Three.js 0.160** + addons: `OrbitControls` (controls), `PLYLoader` (point clouds), `EffectComposer` post-processing pipeline (`UnrealBloomPass`, `RGBShiftShader`).
- All loaded via **importmap** from the unpkg CDN (lines 203–216).
- **CCapture.js** handles offline frame-by-frame export (line 206).
- Renderer enables `preserveDrawingBuffer` (for export), pixel ratio capped at 1.5 (lines 349–351).

---

## Master control panel: `SYSTEM_CONFIG` (lines 249–342)

All tunable parameters live in this single object:

| Section | Contents |
|---------|----------|
| `model` | Point-cloud path, point size/opacity, axis remap, **ceiling-erosion params `ceilingErasure`**, camera start/overhead positions, **12-keyframe cinematic path `autoView.shots`** |
| `data` | Exchange-rate JSON path (`chronicle_data.json`) |
| `export` | Export size 1920×1080, 1800 frames @30fps, timeline segments (intro / dataRun 52s / outro), 4K still settings |

Model currently in use: `models/scaniverse_room_clean_585k.ply` (585k-point dorm room).

---

## The data engine (the heart of the piece)

### 1. Data loading
`fetch(SYSTEM_CONFIG.data.path)` reads `chronicle_data.json` → `exchangeData[]` (line 881).
The data is 25 `{date, rate}` points, GBP/CNY, 2024-08 to 2026-04.

### 2. Rate → stress signals: `getMappedAnxiety(index)` (line 809)
Each rate is converted into normalized (0–1) stress signals:

| Signal | Meaning | Formula |
|--------|---------|---------|
| `deviation` | Distance from baseline | `|rate - 9.00| / 0.72` |
| `velocity` | Volatility / speed of change | `|Δrate| / 0.32` |
| `shock` | Sudden jump | `(|Δrate| - 0.18) / 0.32` |
| **`anxiety`** | **Master scalar** | `deviation×0.62 + velocity×0.30 + shock×0.26` (clamp 0–1) |

`anxiety` is the master switch that drives everything.

### 3. Irreversible damage: `getDeterministicErosionMemory(progress)` (line 840)
On top of anxiety, this accumulates a **"memory / damage" value that mostly only rises**: pressure builds quickly, leaks slowly (`memoryRiseRate` / `memoryFallRate` / `memoryLeakRate`).
It drives the **progressive erosion** of ceiling and walls over time, triggered in stages by the `erosionGate` progress thresholds (objects → structure).

### 4. Key tuning constants (lines 640–646)
```js
const BASELINE_RATE = 9.00;              // baseline rate
const RATE_RANGE_FOR_DEVIATION = 0.72;   // deviation normalization range
const RATE_RANGE_FOR_VELOCITY = 0.32;    // volatility normalization range
const SHOCK_THRESHOLD = 0.18;            // shock trigger threshold
const DEVIATION_WEIGHT = 0.62;           // these three weights decide
const VELOCITY_WEIGHT  = 0.30;           // "how hard the data drives the visuals"
const SHOCK_WEIGHT     = 0.26;
```
> To adjust how strongly the data influences the image, edit here.

---

## Render loop: `animate()` (from line 1513)

Each frame:

1. Compute the playback timeline (intro hold / dataRun / outro) and read current `anxiety`.
2. **Per-vertex point-cloud deformation** (from line 1677): base instability + pressure field + ceiling/wall erosion.
3. **Camera**: three modes — `autoView` cinematic path (12 keyframes interpolated by progress) / OrbitControls auto-rotate / manual (taken over by wheel or drag).
4. **Post-processing**: RGB glitch triggers when `anxiety > 0.7` (line 1663); bloom strength and fog density shift with `endFade`.
5. **Point material**: size and opacity adjust with anxiety.

---

## Sound system (Web Audio, lines 226–238, from 937)

A purely generative soundscape, modulated in real time by `anxiety / shock / movement`:
- Sustained low oscillators (`oscillator` + `subOscillator`)
- Three noise layers (room / event / air)
- Heartbeat clicks (`triggerDataClick`), overload glitches (`triggerOverloadGlitch`)

`tools/generate_lumen_audio.js` is a **standalone Node script** that renders the submission WAV offline (`exports/audio/`).

---

## Export pipeline

- **JPG sequence**: triggered by keyboard shortcut (`keydown`, line 1375); CCapture exports 1920×1080@30fps, 1800 frames (52 s), with title and data panel overlaid (`composeExportFrame`, line 425).
- **4K still**: `startStillExport` family, 3840×2160.
- During export, `virtualTime` replaces real time to guarantee deterministic frame rate.

---

## UI & debug

- **Intro overlay**: click to enter (line 1037).
- **Bottom UI panel**: shows date / rate / stress status.
- **Debug panel** (`buildDebugPanel`, line 1159): camera-keyframe sliders + coordinate capture/copy, for hand-tuning the cinematic path.

---

## File map

| File | Role |
|------|------|
| `index.html` | All runtime logic (WebGL + data + audio + export + UI) |
| `chronicle_data.json` | 25 GBP/CNY rate points (the driving data) |
| `models/*.ply` | Three point clouds (300k / 500k / 585k); currently using 585k |
| `tools/generate_lumen_audio.js` | Node script that renders the submission WAV offline |
| `exports/audio/` | Exported audio deliverables |
| `portfolio*.html` | Portfolio pages |
| `assets/system-diagrams/` | System-as-method diagrams (PDF/PNG/SVG) |
| `*.md` guides | Model replacement, export pipeline, version management, etc. |

---

## Quick reference: common change points

| What you want to do | Where to edit |
|---------------------|---------------|
| Tune data-drive strength | Mapping constants, lines 640–646 |
| Swap the point-cloud model | `SYSTEM_CONFIG.model.path` (line 252) + see `MODEL_REPLACEMENT_GUIDE.md` |
| Change the camera path | `SYSTEM_CONFIG.model.autoView.shots` (line 308) |
| Adjust ceiling/wall erosion pacing | `ceilingErasure` + `erosionGate` (lines 267–299) |
| Change export size/duration | `SYSTEM_CONFIG.export` (lines 329–341) |
| Tune the sound | `initSoundSystem` / `updateDataDrivenSound` (lines 948, 1451) |

---

*Compiled from a full read-through of the code; line numbers reflect the current `index.html`. Keep this in sync after any major refactor.*
