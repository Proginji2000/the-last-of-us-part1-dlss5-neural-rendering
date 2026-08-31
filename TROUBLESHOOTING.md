# Troubleshooting

This file documents the failure modes actually encountered while building the working The Last of Us Part I + OptiScaler + ReShade + DLSS 5 Neural Rendering configuration.

The most important rule is simple:

> **Change one component at a time and test after every change.**

Do not replace a whole stack of DLLs at once.

---

## 1. `0xc0000142` immediately after installing ReShade

### Symptom

The game only launches if ReShade's `dxgi.dll` is renamed/disabled.

### Cause observed in testing

Direct ReShade injection through `dxgi.dll` conflicted with the existing game/mod loader path.

### Working solution

Rename ReShade's DLL:

```text
dxgi.dll
→ ReShade64.dll
```

Then let OptiScaler own `dxgi.dll` and chain-load ReShade:

```ini
[Plugins]
LoadReshade = true
```

Result:

```text
dxgi.dll       = OptiScaler
ReShade64.dll  = ReShade
```

---

## 2. Game works until the RenoDX `.addon64` is present

### Symptom

- ReShade alone works.
- `nvngx_dlssnr.dll` alone works.
- Adding `renodx-dlss5.addon64` causes a crash during startup/loading or when DLSS initializes.

### Useful isolation sequence

Test in this exact order:

```text
A. OptiScaler + ReShade
B. + nvngx_dlssnr.dll
C. + renodx-dlss5.addon64
```

If A and B work but C fails, the problem is not simply the DLSSNR DLL.

---

## 3. `Failed to find slSetTagForFrame`

A failing test produced:

```text
[DLSS 5 Neural Rendering] vtable::Hook(Failed to find slSetTagForFrame)
```

The same run could still hook older Streamline functions such as:

```text
slEvaluateFeature
slSetTag
```

### Interpretation

The experimental add-on is seeing a Streamline interface that does not expose the newer function it expects.

### Working workaround

Do not replace every Streamline DLL.

Instead rename only:

```text
sl.common.dll
→ sl.common.dll.bak
```

Then use the game's **FSR 3.1 / FFX-DX12** path as OptiScaler input and convert that path to DLSS.

---

## 4. `Failed to find NVSDK_NGX_D3D12_EvaluateFeature_C`

The tested log also showed:

```text
Failed to find NVSDK_NGX_D3D12_EvaluateFeature_C
```

while immediately afterwards successfully installing hooks for:

```text
NVSDK_NGX_D3D12_CreateFeature
NVSDK_NGX_D3D12_EvaluateFeature
NVSDK_NGX_D3D12_ReleaseFeature
```

and reporting:

```text
DLSS5 Generic: D3D12 NGX hooks installed
```

### Meaning for this guide

This message alone did **not** prevent the final configuration from working.

The final proof of success is not the absence of every error line; it is:

```text
DLSSNR ... ACTIVE
Successful NR frames: increasing
```

---

## 5. Add-on appears in ReShade, but Neural Rendering is not actually working

Do not treat the checkbox/menu entry as proof.

Open:

```text
ReShade
→ Add-ons
→ DLSS 5 Neural Rendering
```

Look for:

```text
DLSSNR v310.8.0: ACTIVE
Successful NR frames: [increasing]
```

If the counter does not increase, the add-on may be loaded but not processing the game's frames.

---

## 6. ReShade disappears after enabling XeFG

### Symptom

- XeFG works and the displayed FPS roughly doubles.
- ReShade's overlay no longer appears.
- DLSS 5 Neural Rendering is no longer verifiably active on the final display path.

### Key ReShade log message

```text
Skipping swap chain because it was created without a proxy Direct3D device.
```

### Working solution

Use:

```text
FG Input  = FSR 3.1 FG
FG Output = FSR FG
```

instead of XeFG for the combined ReShade/DLSS5 configuration.

This allowed both FSR Frame Generation and DLSSNR to remain active.

---

## 7. Frame Generation is selected in OptiScaler but not active

Make sure the game itself is using:

```text
AMD FSR 3.1
Frame Generation: ON
```

Then in OptiScaler:

```text
FG Input  = FSR 3.1 FG
FG Output = FSR FG
```

Also disable:

```text
Options
→ Accessibility
→ Motion Sickness
→ Full-screen effects
```

The OptiScaler TLOU Part I compatibility page documents a HUDless issue caused by the game's full-screen/low-health effects.

---

## 8. RenoDX `Enable Upscaling` gives much better FPS but the image flickers

### Observed result

With RenoDX internal upscaling enabled, the tested system rose from roughly:

```text
~58–62 rendered FPS
```

to roughly:

```text
~84 rendered FPS
```

and approximately:

```text
~168 displayed FPS with FSR FG
```

However, the image showed severe temporal shimmering/flicker in motion.

### Recommendation

Use:

```text
Enable Upscaling = OFF
```

The normal full-resolution Neural Rendering path was much more temporally stable.

Do not randomly change motion-vector or depth-convention debug overrides trying to hide the shimmer; incorrect guide metadata can make temporal artifacts much worse.

---

## 9. Startup crash after replacing the full Streamline stack

### What happened

A test copied an entire newer package containing multiple files such as:

```text
sl.common.dll
sl.dlss.dll
sl.dlss_g.dll
sl.dlss_nr.dll
sl.interposer.dll
sl.nis.dll
sl.pcl.dll
...
```

over the game's files.

The game then crashed at startup.

### Recommendation

**Do not do this.**

The working setup does not require replacing the whole Streamline stack.

Use the reversible rename:

```text
sl.common.dll
→ sl.common.dll.bak
```

instead.

---

## 10. No `sl.reflex.dll` in the game folder

That is fine.

The tested clean installation did not contain one either.

Do not copy a DLL into the game merely because another guide lists it. Work from the files that actually exist in your own clean install.

---

## 11. ReShade is not visible — how to tell whether it loaded

Rename an existing log before launching:

```text
ReShade.log
→ ReShade.log.old
```

Launch the game, wait a few seconds, then close it.

If a new `ReShade.log` appears and begins with something similar to:

```text
Initializing crosire's ReShade ... loaded from ... ReShade64.dll
```

then OptiScaler is chain-loading ReShade successfully.

If the overlay is still missing, inspect the log for swap-chain messages rather than reinstalling ReShade immediately.

---

## 12. The game crashes while changing upscaling/FG options

The TLOU Part I OptiScaler compatibility notes mention that enabling FG can itself cause a crash on this game and that the next boot may work normally.

Keep a reversible configuration and restart before assuming the installation is corrupt.

---

## Diagnostic checklist

If you need to report a problem, collect:

```text
ReShade.log
OptiScaler.log (enable logging if needed)
OptiScaler.ini
GPU + driver version
Game build/version
Exact contents of the game folder
The exact step that starts the crash
```

Also state whether each checkpoint works:

```text
Vanilla game                         YES / NO
OptiScaler only                      YES / NO
OptiScaler + ReShade                 YES / NO
+ nvngx_dlssnr.dll                   YES / NO
+ renodx-dlss5.addon64               YES / NO
sl.common.dll renamed                YES / NO
FFX-DX12 → DLSS                      YES / NO
FSR FG                               YES / NO
DLSSNR ACTIVE / frames increasing    YES / NO
```
