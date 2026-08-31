# The Last of Us Part I — DLSS 5 Neural Rendering (Experimental Guide)

> **Status:** experimental, unofficial, community-tested configuration.
>
> This repository documents a configuration that was successfully tested on **The Last of Us Part I (PC)** with **DLSS 5 Neural Rendering active**, **DLSS Super Resolution via OptiScaler**, and **FSR 3.1 Frame Generation** working at the same time.
>
> It is **not affiliated with or endorsed by NVIDIA, Naughty Dog, Sony, RenoDX, ReShade, OptiScaler, Nexus Mods, or any of their contributors.**

## What this guide achieves

The working pipeline tested here is:

```text
The Last of Us Part I
        ↓
FSR 3.1 / FFX-DX12 input
        ↓
OptiScaler 0.9.4
        ↓
DLSS Super Resolution (NGX)
        ↓
DLSS 5 Neural Rendering (RenoDX add-on)
        ↓
FSR 3.1 Frame Generation
        ↓
Display
```

The key workaround is to **disable the game's old native Streamline path** and use the FSR 3.1 / FFX-DX12 path as OptiScaler input instead.

## Tested configuration

| Component | Tested value |
|---|---|
| GPU | NVIDIA GeForce RTX 5080 |
| Resolution | 3440×1440 |
| ReShade | 6.8.0 Full Add-On Support |
| OptiScaler | 0.9.4-final |
| RenoDX DLSS 5 add-on | `DLSS 5 Neural Rendering` v0.2026.827.2036 |
| DLSSNR | v310.8.0 |
| API | DirectX 12 |
| OptiScaler input | FFX-DX12 / FSR 3.1 |
| OptiScaler output | DLSS |
| FG input | FSR 3.1 FG |
| FG output | FSR FG |

Other GPUs, game builds, drivers, or later add-on versions may behave differently.

---

# 1. Downloads

Use the original project/download pages. **This repository intentionally does not redistribute third-party DLLs or add-ons.**

- **ReShade:** https://reshade.me/
  - Use the build with **Full Add-On Support**.
- **OptiScaler:** https://github.com/optiscaler/OptiScaler/releases
  - Tested here with **v0.9.4**.
- **OptiScaler — TLOU Part I compatibility notes:** https://github.com/optiscaler/OptiScaler/wiki/The-Last-of-Us-Part-I
- **Experimental RenoDX / DLSS 5 multi-game files/guide:** https://www.nexusmods.com/site/mods/2224

> The DLSS 5 files are experimental. Check the current file status, scan results, comments, and version notes before downloading anything.

---

# 2. Back up the clean game first

Before installing anything, make a copy of the original DLLs in the game folder.

At minimum, back up every existing `sl.*.dll` plus:

```text
nvngx_dlss.dll
```

Do **not** assume a DLL exists just because it appears in another game's guide. For example, the tested installation did **not** contain `sl.reflex.dll`.

A full copy of the game folder is recommended if your game build is difficult to restore.

---

# 3. Verify the vanilla baseline

Launch the game before modding it.

Confirm that:

- the game reaches the menu;
- DLSS can be enabled normally;
- a save can be loaded;
- the game is stable.

Do not continue until the vanilla baseline works.

---

# 4. Install ReShade 6.8 Full Add-On Support

Run the ReShade installer and select:

```text
tlou-i.exe
```

Choose:

```text
DirectX 10/11/12
```

Shader/effect packages are not required for this guide.

## Important: do not leave ReShade as `dxgi.dll`

On the tested setup, direct ReShade injection as `dxgi.dll` caused:

```text
0xc0000142
```

Instead, rename the ReShade DLL created by the installer:

```text
dxgi.dll
→ ReShade64.dll
```

ReShade will later be chain-loaded by OptiScaler.

---

# 5. Install OptiScaler 0.9.4

Install OptiScaler next to `tlou-i.exe`.

For The Last of Us Part I, use:

```text
dxgi.dll
```

as the OptiScaler loader/proxy.

This matches the game's OptiScaler compatibility entry.

Your folder should now contain, among other game files:

```text
tlou-i.exe

dxgi.dll          ← OptiScaler
OptiScaler.ini

ReShade64.dll     ← ReShade
ReShade.ini
```

Open `OptiScaler.ini` and make sure the following is enabled:

```ini
[Plugins]
LoadReshade = true
```

## Checkpoint

Before installing DLSS 5, launch the game and confirm:

```text
OptiScaler 0.9.4   ✅
ReShade 6.8        ✅
Normal gameplay    ✅
```

---

# 6. Add DLSSNR first

Copy only:

```text
nvngx_dlssnr.dll
```

next to `tlou-i.exe`.

Launch the game again and load a save.

On the tested configuration, **`nvngx_dlssnr.dll` alone did not cause a crash**.

This is a useful isolation test before adding the RenoDX add-on.

---

# 7. Add the RenoDX DLSS 5 add-on

Copy:

```text
renodx-dlss5.addon64
```

next to `tlou-i.exe`.

Then edit `ReShade.ini`.

If a `[ADDON]` section already exists, add the line to that existing section. Do not create duplicate `[ADDON]` sections.

```ini
[ADDON]
LoadFromDllMain=renodx-dlss5.addon64
```

This forces the add-on to load early.

At this stage the add-on can load, but on the game's native Streamline path it may still crash or report missing hooks. The next section is the critical workaround.

---

# 8. Critical workaround: disable native Streamline

Close the game.

Rename the game's original:

```text
sl.common.dll
→ sl.common.dll.bak
```

**Do not delete it.**

This disables the game's native Streamline path (and therefore its native DLSS path), which is important because the tested native Streamline integration does not expose everything the experimental DLSS 5 add-on expects.

The OptiScaler TLOU Part I compatibility guide also documents renaming `sl.common.dll` when using the FSR 3.1 path.

---

# 9. Configure the game and OptiScaler

Launch the game.

## In The Last of Us Part I

Select:

```text
Upscaling: AMD FSR 3.1
Quality mode: Quality
Frame Generation: ON
```

## In OptiScaler

Set:

```text
Upscaler Input : FFX-DX12 / FSR 3.1
Upscaler Output: DLSS

FG Input : FSR 3.1 FG
FG Output: FSR FG
```

Save the INI/settings and restart the game if requested.

### Accessibility setting required for FG stability

Go to:

```text
Options
→ Accessibility
→ Motion Sickness
→ Full-screen effects: OFF
```

OptiScaler documents a TLOU Part I issue where low-health/full-screen effects can contaminate the HUDless resource and interfere with frame generation.

---

# 10. Verify that DLSS 5 is really running

Open ReShade with the **Home** key and go to:

```text
Add-ons
→ DLSS 5 Neural Rendering
```

Do not assume it works merely because the add-on appears in the menu.

The tested working state showed:

```text
DLSSNR v310.8.0: ACTIVE
Backend: NGX core
Successful NR frames: [increasing]
NGX evaluations: [increasing]
```

A continuously increasing `Successful NR frames` counter is the most useful confirmation that Neural Rendering is actually processing frames.

The tested pipeline also reported that insertion happened immediately after the NGX DLSS output.

---

# 11. Recommended RenoDX settings used during testing

These are not mandatory; they are simply a reasonable starting point:

```text
NR Intensity              0.80
Local Tone Strength       0.75
Local Structure Strength  0.75
Skin Structure Strength  -0.51
Automatic Mask            ON
Enable Upscaling           OFF
```

The intensity sliders primarily alter the visual result. They did not solve the major performance cost of Neural Rendering in this test.

---

# 12. Frame Generation: use FSR FG, not XeFG, for this combined setup

`XeFG` itself worked with OptiScaler and produced very high displayed FPS, but with this ReShade/DLSS5 chain ReShade stopped attaching to the final swap chain.

The ReShade log showed:

```text
Skipping swap chain because it was created without a proxy Direct3D device.
```

As a result, the ReShade overlay disappeared and DLSS 5 Neural Rendering was no longer confirmed active on the displayed pipeline.

The working combined configuration was therefore:

```text
FG Input  = FSR 3.1 FG
FG Output = FSR FG
```

This allowed both:

```text
DLSSNR ACTIVE ✅
FSR Frame Generation ACTIVE ✅
```

to coexist.

---

# 13. Performance observed at 3440×1440

These measurements are from **one RTX 5080 setup in one test scene**. They are not universal benchmarks.

| DLSS mode | Approx. internal resolution | Rendered FPS | Displayed FPS with FSR FG |
|---|---:|---:|---:|
| Quality | 2288×960 | ~57.9 | ~115.9 |
| Balanced | 2016×840 | ~60.6 | ~121.3 |
| Performance | 1720×720 | ~61.8 | ~123.6 |

The small difference between Quality and Performance strongly suggests that the expensive stage in this configuration is the **full-resolution DLSS 5 Neural Rendering pass**, not DLSS Super Resolution itself.

For that reason, **Quality** was the preferred setting: substantially better source quality for only a few FPS less.

## RenoDX `Enable Upscaling`

Enabling RenoDX's own upscaling produced roughly:

```text
~84 rendered FPS
~168 displayed FPS with FG
```

but caused severe temporal shimmering/flickering in motion.

Therefore the tested recommendation is:

```text
Enable Upscaling = OFF
```

until a newer add-on/game integration improves temporal stability.

---

# 14. What NOT to do

## Do not replace the entire Streamline stack

A test replacing the game's complete `sl.*.dll` stack with files from a newer experimental package caused the game to crash at startup.

Do not blindly overwrite files such as:

```text
sl.common.dll
sl.dlss.dll
sl.interposer.dll
sl.nis.dll
sl.pcl.dll
...
```

The working method only requires **renaming the original `sl.common.dll`** so it can easily be restored.

## Do not copy every DLL from a DLSS 5 archive

For the tested working configuration, the relevant experimental additions were only:

```text
nvngx_dlssnr.dll
renodx-dlss5.addon64
```

alongside ReShade and OptiScaler.

---

# 15. Fast vanilla rollback — delete nothing

To return to the native game without uninstalling any mod files, close the game and rename:

```text
dxgi.dll
→ dxgi.dll.OFF

sl.common.dll.bak
→ sl.common.dll

renodx-dlss5.addon64
→ renodx-dlss5.addon64.OFF

nvngx_dlssnr.dll
→ nvngx_dlssnr.dll.OFF
```

You may leave these files in place:

```text
ReShade64.dll
ReShade.ini
OptiScaler.ini
reshade-shaders\
```

Without the OptiScaler loader, the tested ReShade chain-loader is inactive.

To re-enable the experimental setup later, reverse the four renames.

See [ROLLBACK.md](ROLLBACK.md) for the full procedure.

---

# Troubleshooting

See **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** for the failure modes encountered during testing, including:

- `0xc0000142` on startup;
- crash when enabling native DLSS;
- `Failed to find slSetTagForFrame`;
- `Failed to find NVSDK_NGX_D3D12_EvaluateFeature_C`;
- ReShade disappearing when using XeFG;
- Neural Rendering loaded but not active;
- severe flicker with RenoDX upscaling;
- startup failure after replacing Streamline DLLs.

For performance notes, see **[PERFORMANCE.md](PERFORMANCE.md)**.

---

# Credits / upstream projects

This repository contains **documentation only** and does not redistribute third-party binaries.

Credit belongs to the upstream projects and developers, including:

- **ReShade / crosire:** https://reshade.me/
- **OptiScaler contributors:** https://github.com/optiscaler/OptiScaler
- **RenoDX / ShortFuse and related contributors:** https://github.com/clshortfuse/renodx
- **Experimental DLSS 5 multi-game guide/files:** https://www.nexusmods.com/site/mods/2224

Please respect the licenses and redistribution rules of every upstream project and download page.

---

# Disclaimer

This is an experimental configuration built around unofficial/early neural-rendering files and third-party injection/modding tools. It may crash, corrupt settings, produce visual artifacts, or stop working after a game/driver/tool update.

Back up your files first. Use at your own risk.
