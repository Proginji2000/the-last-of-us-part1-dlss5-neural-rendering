# Upstream sources and references

This repository contains **documentation only**.

It does not redistribute NVIDIA DLLs, RenoDX add-ons, ReShade binaries, OptiScaler binaries, or game files.

## ReShade

Official site:

https://reshade.me/

Use the version with **Full Add-On Support** for this guide.

## OptiScaler

Repository:

https://github.com/optiscaler/OptiScaler

Releases:

https://github.com/optiscaler/OptiScaler/releases

The Last of Us Part I compatibility notes:

https://github.com/optiscaler/OptiScaler/wiki/The-Last-of-Us-Part-I

The compatibility page documents, among other things:

- `dxgi.dll` as the tested loader filename;
- DLSS and FSR 3.1 + FG as inputs;
- `FSR 3.1 FG → XeFG/FSR-FG`;
- renaming `sl.common.dll` to disable Streamline when using the FSR 3.1 path;
- disabling full-screen motion-sickness effects to avoid HUDless/FG issues.

## RenoDX

Project:

https://github.com/clshortfuse/renodx

The experimental add-on used in this guide identified itself during testing as:

```text
DLSS 5 Neural Rendering
v0.2026.827.2036
```

## Experimental DLSS 5 multi-game guide/files

Nexus Mods page used during testing:

https://www.nexusmods.com/site/mods/2224

The page is explicitly experimental and multi-game. File availability, scan status, names, and compatibility may change rapidly.

Always review the latest page status before downloading anything.

## NVIDIA / DLSS

This repository does not provide NVIDIA binaries.

The tested experimental Neural Rendering library reported:

```text
nvngx_dlssnr.dll
DLSSNR v310.8.0
```

Users are responsible for sourcing any NVIDIA-related file from a source they trust and for respecting applicable licenses/redistribution restrictions.

---

# Empirical vs upstream information

The guide deliberately separates two types of information:

## Upstream-documented

Examples:

- OptiScaler compatibility recommendations for TLOU Part I;
- ReShade Full Add-On requirement;
- available OptiScaler FG input/output modes.

## Empirically observed in this test session

Examples:

- direct ReShade `dxgi.dll` causing `0xc0000142` on the tested setup;
- `nvngx_dlssnr.dll` loading safely by itself;
- RenoDX reporting missing `slSetTagForFrame` on the native Streamline path;
- the successful FFX-DX12 → DLSS → DLSS5 NR pipeline;
- XeFG causing ReShade to skip the final swap chain;
- FSR FG allowing DLSSNR and FG to coexist;
- measured performance numbers and RenoDX upscaling shimmer.

Do not assume empirical behavior will reproduce on every driver/game/tool version.
