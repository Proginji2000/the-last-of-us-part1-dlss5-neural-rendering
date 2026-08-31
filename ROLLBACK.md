# Reversible rollback to vanilla

This procedure disables the experimental chain **without deleting files**.

Use it when you want to return to the game's native rendering path while keeping the mod files available for later testing.

## Before starting

Close The Last of Us Part I completely.

---

## Disable OptiScaler

Rename:

```text
dxgi.dll
→ dxgi.dll.OFF
```

In the tested setup, `dxgi.dll` is the OptiScaler loader.

Disabling it also stops OptiScaler from chain-loading `ReShade64.dll`.

---

## Restore native Streamline

Rename:

```text
sl.common.dll.bak
→ sl.common.dll
```

This restores the game's native Streamline/DLSS path.

---

## Disable the experimental DLSS 5 files

Rename:

```text
renodx-dlss5.addon64
→ renodx-dlss5.addon64.OFF
```

and:

```text
nvngx_dlssnr.dll
→ nvngx_dlssnr.dll.OFF
```

---

## Files that can remain untouched

You can leave these files in the folder:

```text
ReShade64.dll
ReShade.ini
ReShadePreset.ini
reshade-shaders\
OptiScaler.ini
```

Because the OptiScaler loader has been disabled, ReShade should no longer be chain-loaded in the tested configuration.

---

## Expected vanilla-like state

The relevant files should look similar to:

```text
dxgi.dll.OFF
sl.common.dll
renodx-dlss5.addon64.OFF
nvngx_dlssnr.dll.OFF
ReShade64.dll
OptiScaler.ini
```

Launch the game and verify:

```text
OptiScaler overlay absent
ReShade overlay absent
DLSS 5 add-on absent
Native NVIDIA DLSS available again
```

---

# Re-enable the experimental configuration

Close the game and reverse the renames:

```text
dxgi.dll.OFF
→ dxgi.dll

sl.common.dll
→ sl.common.dll.bak

renodx-dlss5.addon64.OFF
→ renodx-dlss5.addon64

nvngx_dlssnr.dll.OFF
→ nvngx_dlssnr.dll
```

Then verify that the following are still configured:

## `OptiScaler.ini`

```ini
[Plugins]
LoadReshade = true
```

## `ReShade.ini`

```ini
[ADDON]
LoadFromDllMain=renodx-dlss5.addon64
```

## OptiScaler

```text
Upscaler Input : FFX-DX12 / FSR 3.1
Upscaler Output: DLSS
FG Input        : FSR 3.1 FG
FG Output       : FSR FG
```

## Game

```text
AMD FSR 3.1
Frame Generation ON
```

Then verify in the ReShade add-on panel:

```text
DLSSNR ... ACTIVE
Successful NR frames: increasing
```

---

# If rollback does not restore the game

Do not start replacing DLLs at random.

Compare the game folder with the backup taken before installation.

Priority files to check are:

```text
sl.common.dll
nvngx_dlss.dll
```

and any other `sl.*.dll` that may have been modified outside this guide.

If you previously replaced the complete Streamline stack, restore those original files from your backup or use your game platform's integrity/repair mechanism if available for your build.
