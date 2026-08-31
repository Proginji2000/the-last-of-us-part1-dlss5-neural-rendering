# Performance notes

These measurements are **observations from one tested system**, not universal benchmarks.

## Tested system/context

```text
GPU: NVIDIA GeForce RTX 5080
Resolution: 3440×1440
API: DirectX 12
OptiScaler: 0.9.4-final
ReShade: 6.8.0 Full Add-On Support
DLSSNR: v310.8.0
Neural Rendering: ACTIVE
Frame Generation: FSR 3.1 FG → FSR FG
```

The scene and exact game workload influence all numbers below.

---

## DLSS quality-mode comparison

With full-resolution DLSS 5 Neural Rendering enabled and RenoDX internal upscaling disabled:

| Mode | Approx. internal resolution | Rendered FPS | Displayed FPS with FSR FG |
|---|---:|---:|---:|
| Quality | 2288×960 | ~57.9 | ~115.9 |
| Balanced | 2016×840 | ~60.6 | ~121.3 |
| Performance | 1720×720 | ~61.8 | ~123.6 |

### Interpretation

Dropping from Quality to Performance lowers the DLSS input resolution substantially but only improves rendered FPS by roughly a few frames.

That strongly suggests that, in this configuration, the dominant cost is **the DLSS 5 Neural Rendering pass at/near output resolution**, not the initial DLSS Super Resolution stage.

### Practical recommendation

Use:

```text
DLSS Quality
```

unless you specifically prefer a lower-resolution input.

Quality retained noticeably more source detail while giving up very little performance compared with Balanced/Performance.

---

## Frame Generation

With the working combined path:

```text
FG Input  = FSR 3.1 FG
FG Output = FSR FG
```

roughly 55–62 rendered FPS became approximately 110–124 displayed FPS.

Example observed overlay:

```text
~55.8 rendered FPS
~111.6 displayed FPS
```

This is normal 2× frame-generation behavior.

### Important

Frame Generation improves visual smoothness/display FPS but does not turn a 56 FPS simulation/input latency profile into a true 112 FPS native-rendered profile.

For responsiveness, raising the **base rendered FPS** is still valuable.

---

## XeFG test

XeFG worked by itself with OptiScaler and produced roughly:

```text
~95.7 rendered FPS
~191.5 displayed FPS
```

in one observed state.

However, when combined with this ReShade/RenoDX DLSS5 chain, ReShade stopped attaching to the final swap chain.

The relevant log line was:

```text
Skipping swap chain because it was created without a proxy Direct3D device.
```

Therefore those very high XeFG numbers were **not accepted as a valid DLSS 5 Neural Rendering + FG result**.

For this guide, use **FSR FG** if you want DLSSNR and FG simultaneously.

---

## RenoDX `Enable Upscaling` test

This setting was interesting from a performance perspective.

With RenoDX internal upscaling enabled, the DLSS5 panel reported a lower-resolution guide/input path and output to 3440×1440.

Observed performance rose to approximately:

```text
~84 rendered FPS
~168 displayed FPS with FSR FG
```

This was a major performance improvement.

### Why it is not recommended

The image showed persistent temporal shimmering/flicker, including after changing the game's quality preset.

Therefore the tested stable recommendation is:

```text
Enable Upscaling = OFF
```

until the add-on or game-specific integration improves the temporal guide path.

---

## Neural Rendering strength sliders

The following values were used as a visually reasonable starting point:

```text
NR Intensity              0.80
Local Tone Strength       0.75
Local Structure Strength  0.75
Skin Structure Strength  -0.51
Automatic Mask            ON
```

Reducing these values did **not** appear to offer the sort of large performance gain needed to solve the ~60 FPS base-render issue.

Treat them mainly as image-character controls, not performance controls.

---

## Why unofficial performance should not be used to judge final DLSS 5

This setup is not a native Naughty Dog/NVIDIA integration.

It uses:

- a game not authored for this neural-rendering path;
- ReShade injection;
- RenoDX interception;
- OptiScaler conversion from FSR/FFX input to DLSS;
- a workaround that disables the game's native Streamline path;
- an experimental DLSSNR DLL/add-on.

The overhead and resource flow are therefore not representative of what a future native game integration may achieve.

The fact that the technology works at all in this configuration is more meaningful than treating these FPS numbers as a prediction of official DLSS 5 performance.

---

## Recommended practical profile

For the tested 3440×1440 RTX 5080 setup:

```text
Game input             FSR 3.1 Quality
OptiScaler output      DLSS
DLSS 5 NR              ON
RenoDX upscaling       OFF
FSR 3.1 Frame Gen      ON
OptiScaler FG output   FSR FG
```

Expected ballpark from the tested scene:

```text
~58 rendered FPS
~116 displayed FPS
```

This was the best combination of image quality, temporal stability and working Neural Rendering found during the test session.
