# Working render pipeline

This document explains why the final configuration differs from a normal native-DLSS setup.

## Native TLOU Part I path

Normally, the game can use its own NVIDIA Streamline/DLSS integration:

```text
Game
 ↓
Native Streamline (`sl.common.dll`, etc.)
 ↓
Native DLSS / NGX
 ↓
Display
```

The experimental RenoDX DLSS 5 add-on successfully loaded on this path but reported missing interfaces such as:

```text
Failed to find slSetTagForFrame
```

That path also produced crashes during the test session.

---

## Final working path

The working configuration disables the game's native Streamline loader by renaming:

```text
sl.common.dll
→ sl.common.dll.bak
```

The game is then configured to use AMD FSR 3.1 instead.

OptiScaler intercepts the FFX-DX12 path and outputs NVIDIA DLSS:

```text
The Last of Us Part I
        │
        │ FSR 3.1 / FFX-DX12 calls
        ▼
     OptiScaler
        │
        │ converts to DLSS / NGX
        ▼
 DLSS Super Resolution
        │
        │ RenoDX hooks NGX output
        ▼
DLSS 5 Neural Rendering
        │
        ▼
 FSR 3.1 Frame Generation
        │
        ▼
      Display
```

The DLSS 5 add-on reported a working NGX backend and increasing successful Neural Rendering frames on this path.

---

## Why FSR FG is used for the final stage

OptiScaler's TLOU Part I compatibility notes support:

```text
FSR 3.1 FG → XeFG / FSR-FG
```

XeFG itself worked during testing and approximately doubled displayed FPS, but it created a final swap chain that ReShade did not attach to correctly.

ReShade logged:

```text
Skipping swap chain because it was created without a proxy Direct3D device.
```

Therefore, the final combined pipeline uses:

```text
FG Input  = FSR 3.1 FG
FG Output = FSR FG
```

This kept ReShade, RenoDX DLSS5, and Frame Generation alive simultaneously.

---

## ReShade / OptiScaler ownership

The working DLL ownership is:

```text
dxgi.dll       → OptiScaler loader
ReShade64.dll  → ReShade, chain-loaded by OptiScaler
```

with:

```ini
[Plugins]
LoadReshade = true
```

in `OptiScaler.ini`.

RenoDX is loaded early from ReShade using:

```ini
[ADDON]
LoadFromDllMain=renodx-dlss5.addon64
```

---

## Minimum experimental files added by this guide

The tested successful configuration adds only these DLSS5-specific files:

```text
nvngx_dlssnr.dll
renodx-dlss5.addon64
```

It does **not** require replacing the game's entire Streamline stack.

---

## Verification signals

A correct render path must be validated in the RenoDX/ReShade add-on panel.

Expected indicators include:

```text
DLSSNR v310.8.0: ACTIVE
Backend: NGX core
Successful NR frames: increasing
```

Seeing the add-on in the ReShade UI is not sufficient by itself.
