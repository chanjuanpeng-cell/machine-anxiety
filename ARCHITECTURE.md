# Machine Anxiety — 代码结构说明 (ARCHITECTURE)

> 单文件 vibe-coded WebGL 作品。核心逻辑全部位于 `index.html` 的一个 ES module `<script>` 中(约 1865 行)。
> 本文档梳理「汇率数据如何变成画面与声音」这条主线,并标注关键行号便于定位。

---

## 一句话概括

**汇率 → `anxiety` 标量 → 同时驱动点云变形、摄影机运镜、后期故障、生成式音景** 的实时系统。

```
chronicle_data.json (25 个 GBP/CNY 汇率点)
        │
        ▼
getMappedAnxiety()        ── 把汇率换算成压力信号 (deviation / velocity / shock)
        │                    并加权合成主标量 anxiety
        ▼
getDeterministicErosionMemory()  ── 累积不可逆的「损伤记忆」
        │
        ▼
animate() 每帧驱动 ──┬── 点云逐顶点变形 (含天花板/墙面侵蚀)
                     ├── 摄影机 (电影运镜 / 自转 / 手动)
                     ├── 后期 (Bloom 辉光 / RGB 故障 / 雾)
                     └── Web Audio 生成式音景
```

---

## 技术底座

- **Three.js 0.160** + 扩展:`OrbitControls`(控制器)、`PLYLoader`(点云)、`EffectComposer` 后期管线(`UnrealBloomPass` 辉光、`RGBShiftShader` 故障)。
- 全部通过 **importmap** 从 unpkg CDN 加载(第 203–216 行)。
- **CCapture.js** 负责离线逐帧导出(第 206 行)。
- 渲染器开启 `preserveDrawingBuffer`(用于导出),像素比上限 1.5(第 349–351 行)。

---

## 总控制面板:`SYSTEM_CONFIG`(第 249–342 行)

所有可调参数集中于此对象:

| 区块 | 内容 |
|------|------|
| `model` | 点云路径、点大小/透明度、轴重映射、**天花板侵蚀参数 `ceilingErasure`**、摄影机起始位/俯视位、**12 个关键帧的电影运镜 `autoView.shots`** |
| `data` | 汇率 JSON 路径 (`chronicle_data.json`) |
| `export` | 导出尺寸 1920×1080、1800 帧 @30fps、时长段落 (intro/dataRun 52s/outro)、4K 静帧设置 |

当前使用模型:`models/scaniverse_room_clean_585k.ply`(58.5 万点的宿舍房间)。

---

## 数据引擎(作品的心脏)

### 1. 数据加载
`fetch(SYSTEM_CONFIG.data.path)` 读入 `chronicle_data.json` → `exchangeData[]`(第 881 行)。
数据为 25 个 `{date, rate}` 点,GBP/CNY,2024-08 至 2026-04。

### 2. 汇率 → 压力信号:`getMappedAnxiety(index)`(第 809 行)
每个汇率换算成几个归一化(0–1)的压力信号:

| 信号 | 含义 | 公式 |
|------|------|------|
| `deviation` | 偏离基准的程度 | `|rate - 9.00| / 0.72` |
| `velocity` | 波动速度 | `|Δrate| / 0.32` |
| `shock` | 突跳 | `(|Δrate| - 0.18) / 0.32` |
| **`anxiety`** | **主标量** | `deviation×0.62 + velocity×0.30 + shock×0.26`(clamp 0–1) |

`anxiety` 是驱动一切的总开关。

### 3. 不可逆损伤:`getDeterministicErosionMemory(progress)`(第 840 行)
在 anxiety 之上,逐段累积一个**只增难减的「记忆/损伤」值**:压力累积快、泄漏慢(`memoryRiseRate` / `memoryFallRate` / `memoryLeakRate`)。
它驱动天花板与墙面随时间**逐步被侵蚀消失**,通过 `erosionGate` 的进度阈值分阶段触发(物体 → 结构)。

### 4. 关键调参常量(第 640–646 行)
```js
const BASELINE_RATE = 9.00;              // 基准汇率
const RATE_RANGE_FOR_DEVIATION = 0.72;   // 偏离归一化范围
const RATE_RANGE_FOR_VELOCITY = 0.32;    // 波动归一化范围
const SHOCK_THRESHOLD = 0.18;            // 突跳触发阈值
const DEVIATION_WEIGHT = 0.62;           // 三个权重决定
const VELOCITY_WEIGHT  = 0.30;           // 「数据驱动得多猛」
const SHOCK_WEIGHT     = 0.26;
```
> 想调整数据对画面的影响强度,改这里。

---

## 渲染循环:`animate()`(第 1513 行起)

每帧执行:

1. 计算播放时间线(intro 静置 / dataRun 数据播放 / outro 收尾),取当前 `anxiety`。
2. **点云逐顶点变形**(第 1677 行起):基础不稳定 + 压力场 + 天花板/墙面侵蚀。
3. **摄影机**:三种模式——`autoView` 电影运镜(12 关键帧按进度插值)/ OrbitControls 自转 / 手动(滚轮或拖拽接管)。
4. **后期**:`anxiety > 0.7` 时触发 RGB 故障(第 1663 行);辉光强度、雾浓度随 `endFade` 变化。
5. **点材质**:大小与透明度随 anxiety 调整。

---

## 声音系统(Web Audio,第 226–238、937 行起)

纯生成式音景,实时由 `anxiety / shock / movement` 调制:
- 振荡器持续低音 (`oscillator` + `subOscillator`)
- 三层噪声(房间 `room` / 事件 `event` / 空气 `air`)
- 心跳点击 (`triggerDataClick`)、过载故障 (`triggerOverloadGlitch`)

`tools/generate_lumen_audio.js` 是**独立的 Node 脚本**,离线渲染提交用的 WAV(`exports/audio/`)。

---

## 导出管线

- **JPG 序列**:键盘快捷键触发(`keydown`,第 1375 行),CCapture 导出 1920×1080@30fps、1800 帧(52 秒),叠加标题与数据面板(`composeExportFrame`,第 425 行)。
- **4K 静帧**:`startStillExport` 系列,3840×2160。
- 导出时用 `virtualTime` 替代真实时间,保证帧率确定性。

---

## UI 与调试

- **intro 覆盖层**:点击进入(第 1037 行)。
- **底部 UI 面板**:显示日期 / 汇率 / 压力状态(date / rate / stress)。
- **调试面板**(第 1159 行 `buildDebugPanel`):摄影机关键帧滑块 + 坐标捕获/复制,方便手调运镜。

---

## 文件清单

| 文件 | 作用 |
|------|------|
| `index.html` | 全部运行时逻辑(WebGL + 数据 + 音频 + 导出 + UI) |
| `chronicle_data.json` | 25 个 GBP/CNY 汇率点(驱动数据) |
| `models/*.ply` | 三个点云(30万 / 50万 / 58.5万),当前用 585k |
| `tools/generate_lumen_audio.js` | 离线渲染提交 WAV 的 Node 脚本 |
| `exports/audio/` | 导出的音频成品 |
| `portfolio*.html` | 作品集页面 |
| `assets/system-diagrams/` | 系统方法图(PDF/PNG/SVG) |
| `*.md` 指南 | 模型替换、导出流程、版本管理等文档 |

---

## 常见改动入口速查

| 想做的事 | 改哪里 |
|----------|--------|
| 调数据驱动强度 | 第 640–646 行的映射常量 |
| 换点云模型 | `SYSTEM_CONFIG.model.path`(第 252 行)+ 参考 `MODEL_REPLACEMENT_GUIDE.md` |
| 改运镜 | `SYSTEM_CONFIG.model.autoView.shots`(第 308 行) |
| 调天花板/墙面侵蚀节奏 | `ceilingErasure` + `erosionGate`(第 267–299 行) |
| 改导出尺寸/时长 | `SYSTEM_CONFIG.export`(第 329–341 行) |
| 调声音 | `initSoundSystem` / `updateDataDrivenSound`(第 948、1451 行) |

---

*本文档由代码通读整理而成,行号基于当前 `index.html`。后续若大幅重构,记得同步更新。*
