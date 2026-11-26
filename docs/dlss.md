# DLSS (optional, NVIDIA only)

FSR (AMD's upscaler) stays the default and works on every card. DLSS Super
Resolution is opt in:

```
VICTORY_UPSCALER=dlss
VICTORY_DLSS=quality|balanced|performance|ultra
```

`VICTORY_DLSS` also picks the scene scale the title renders at (2/3, 1/1.7,
1/2, 1/3 of 4K) unless `VICTORY_FSR_4K` is set. If DLSS cannot start (no
NVIDIA card, no DLL, driver too old) a bilinear upscaler serves instead.

## Nothing from NVIDIA is built in

This build links no NVIDIA library and needs no SDK. At run time it loads:

* the NGX core of the NVIDIA display driver: `_nvngx.dll` / `nvngx.dll`
  (found through `HKLM\SOFTWARE\NVIDIA Corporation\Global\NGXCore`, System32,
  or the driver store; `VICTORY_NGX_CORE` names one). It is this DLL, not
  `nvngx_dlss.dll`, that exports the `NVSDK_NGX_D3D12_*` entry points
  (checked on a current driver: `_nvngx.dll` exports `Init_ProjectID`,
  `GetCapabilityParameters`, `CreateFeature`, `EvaluateFeature`,
  `ReleaseFeature`, `DestroyParameters`, `Shutdown1`).
* `nvngx_dlss.dll`, the feature itself, from `VICTORY_DLSS_DLL_DIR` or
  `<folder of victory.exe>\dlss`. `tools/Setup-Dlss.ps1` fetches it from
  NVIDIA's public DLSS repository, checks its SHA-256 and installs it. The
  folder is handed to the core as its feature search path.

NGX parameter blocks are C++ objects; the SDK's helper functions are inline
vtable calls, so they are made through the vtable here, after a write and
read back at start that disables DLSS if the layout is not the expected one.

## Environment

| Variable | Meaning |
| --- | --- |
| `VICTORY_UPSCALER` | `dlss`, `spatial`, or unset for AMD FSR (default) |
| `VICTORY_DLSS` | quality, balanced, performance, ultra |
| `VICTORY_DLSS_DLL_DIR` | folder with `nvngx_dlss.dll` |
| `VICTORY_NGX_CORE` | full path of a specific `_nvngx.dll` / `nvngx.dll` |
| `VICTORY_DLSS_LOG=1` | NGX verbose log into the host log |
