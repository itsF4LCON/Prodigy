# Prodigy

**Prodigy** is an Xbox Series X|S (GDK) → Windows compatibility layer that runs unpacked console games you own natively on x64 Windows—**including on older GPUs without hardware Mesh Shaders** (such as NVIDIA GTX 10-series and AMD RX 500 / 5000). There is no CPU emulation: console system libraries (`kernelx.dll`, `XGameRuntime`, `d3d12_xs.dll`, etc.) are implemented in Rust, and RDNA2 GPU shaders are recompiled to `HLSL` → `DXIL` with automatic Vertex Shader mesh emulation and Wave64-on-Wave32 support.

> Independent continuation of [XCL](https://github.com/ivall/xcl), incorporating select components from [VIctory](https://github.com/Marvisak/VIctory).  
> **Legal:** Works only on unpacked game files you already own. Contains no console SDK code, keys, assets, or DRM circumvention.

## Compatibility

| Game | Status | Notes |
| :--- | :---: | :--- |
| **Minecraft Preview** | ✅ | Playable |
| **Unicorn Overlord** | ✅ | Playable |
| **Fortnite** | ✅ | Boots & runs offline (no multiplayer servers) |
| **XDefiant** | ✅ | Boots & runs offline (official servers shut down) |
| **Unannounced Retail Title** | 🚧 | Boots to menus (~30 FPS); in-game gameplay not reached yet |

## Highlights

- **Older GPU Support:** Automatic Vertex Shader (`vs_6_6`) emulation fallback for console NGG/Mesh/GS pipelines on pre-Mesh-Shader SM 6.6 GPUs (GTX 10-series, RX 500/5000, etc.), plus Wave64-on-Wave32 splitting (`PRODIGY_EMULATE_MESH=1` to force fallback).
- **Native Execution & VFS:** PE loader, `kernelx` memory/threading, DirectStorage (`zlib` + `BCPack`), and a copy-on-write VFS overlay (`GAME:`, `S:\`, `T:\`) that never modifies your game folder.
- **D3D12.X & Shaders:** RDNA2 → DXIL recompiler (VS/PS/CS/GS/Mesh NGG, wave64-on-wave32, ray tracing, disk cache), async PSO creation, GFX10 texture deswizzling, `XboxBvhBuilder`, and FSR 4.
- **Audio & Video:** XAudio2.9 routing, pure-Rust XMA1/XMA2, Opus (`MSOpusDecoder.dll` / `libopus` / built-in `opus-pure` fallback), Bink 2, and a local-only Media Foundation source resolver proxy.
- **Input:** XInput controllers out of the box, plus automatic keyboard & mouse gamepad emulation (`keyboard_mouse = false` in `prodigy.toml` to disable).

## Requirements & Build

- **Windows 10 (22H2+) / Windows 11 x64**, Rust (`stable-x86_64-pc-windows-msvc`)
- **D3D12 GPU** with Shader Model 6.6 (older pre-Mesh-Shader GPUs such as NVIDIA GTX 10-series / AMD RX 500 & 5000 automatically use the Vertex Shader mesh-emulation fallback; Windows 10: place Agility SDK `D3D12Core.dll` in `D3D12\` next to `prodigy.exe` or set `PRODIGY_D3D12_DIR`)
- **DXC 1.8+** (`dxcompiler.dll` + `dxil.dll`, set `PRODIGY_DXC_DIR` to `bin\x64`)

```powershell
cargo build --release --workspace
```

## Usage

Double-click `target\release\prodigy.exe` for the GUI picker, or use the CLI:

```powershell
prodigy check [--json]
prodigy run D:\Games\MyTitle [-- game args]
prodigy inspect D:\Games\MyTitle\game.exe
prodigy disasm <shader.bin>
prodigy shaders <shaders.rpf> --top 50
```

Per-title config, saves, overlay writes, shader caches, and `logs\report.json` live in `%LOCALAPPDATA%\Prodigy\<title>\` (override with `PRODIGY_DATA_ROOT`).

## Tests

```powershell
.\scripts\test.ps1
```

Use `PRODIGY_ADAPTER=<index|name|warp>` to pick a GPU/WARP adapter, or `PRODIGY_REQUIRE_GPU=wave64,mesh` to fail instead of skipping hardware-gated tests.

## License

Licensed under **GPL-2.0-only** — see [`LICENSE`](LICENSE) and [`THIRD_PARTY.md`](THIRD_PARTY.md).
