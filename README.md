# Prodigy – Xbox Series X|S → Windows Compatibility Layer

> **Note:** This repository is an independent spin/continuation of [XCL](https://github.com/ivall/xcl).

**Prodigy** is a compatibility layer that runs Xbox Series X|S (GDK) games you own natively on a Windows PC—in the spirit of Wine/Proton. There is no CPU emulation: the game's x64 code executes directly on your processor, while Prodigy provides independently written implementations of console-exclusive system libraries (`kernelx.dll`, `XGameRuntime`, `d3d12_xs.dll`, etc.). Console GPU shaders (RDNA2 machine code) are statically recompiled into PC shaders (`HLSL` → `DXIL`).

### Legal & Scope Disclaimer
* **Bring Your Own Games:** This project operates strictly on unpacked game files you already own. It does not download, decrypt, or unpack console packages (`XVC` / `MSIXVC`) and contains zero functionality for circumventing DRM.
* **Clean-Room Implementation:** This repository contains no source code, headers, keys, or binaries from any console SDK, nor any copyrighted game assets. Standard Windows components (`XGameRuntime`, `XAudio2`, `D3D12`, `DXC`) are loaded from your local OS or the public Windows SDK.

---

## Compatibility Status

| Game | Package (Version) | State | What Works | Current Limits |
| :--- | :--- | :---: | :--- | :--- |
| **Minecraft Preview** | `Microsoft.MinecraftBeta` 1.18.3064.0 | ✅ | Fully playable | – |
| **Unicorn Overlord** | `SEGAofAmericaInc.Pa253b0c1Y` 1.0.0.6 | ✅ | Fully playable | – |
| **Fortnite** | `436609B6.FortniteClient` 2.0.1461.7919 | ✅ | Boots and runs as far as offline mode permits | Online-only multiplayer servers |
| **XDefiant** | `Ubisoft.XDefiant-Game` 1.0.0.186 | ✅ | Boots and runs as far as offline mode permits | Official servers are shut down |
| **Unannounced Retail Title** | – | 🚧 | Boots in ~4s, steady 30 FPS in menus, navigates to settings (~15 GB RAM committed) | In-game gameplay not reached yet; minimap mesh pipelines currently refused |

---

## Architecture & Capabilities

### Core Execution & OS Layer
* **PE Loader:** Handles relocations, name/ordinal/delay-load imports, TLS callbacks, exception tables, and data-driven import routing (HLE / Windows / stub).
* **`kernelx` & Memory:** Emulates console title memory budgets (`MicrosoftGame.Config`), physical page allocations, thread scheduling, and shared CPU/GPU title graphics memory up to 16 GB (4 GiB windows) without redundant copies.
* **Virtual File System:** Maps console paths (`GAME:`, `G:`, `D:`, etc.) to your local game directory using a copy-on-write overlay—**your original game files are never modified**. Routes persistent local storage (`S:\`) and temp storage (`T:\`) to `%LOCALAPPDATA%`.
* **DirectStorage:** Supports console file/memory queues, `zlib`, and `BCPack` texture decompression (`BC1`, `BC3`, `BC4`, `BC5`, `BC6H`, `BC7`).

### Graphics & Media Pipeline
* **RDNA2 → DXIL Shader Recompiler:** Translates Vertex, Pixel, Compute, Geometry, and Mesh (NGG) shaders—including atomics, LDS, wave ops, static samplers, and ray-tracing instructions—via parallel DXC compilation with persistent disk caching.
* **D3D12.X Graphics & Compute:** Replays console command lists, descriptor heaps (`T#` / `V#` / `S#`), root arguments, and `PresentX` frame pacing. Non-blocking PSO creation builds pipelines asynchronously in the background.
* **Texture Deswizzling:** Supports GFX10 tiling (all 14 2D swizzle modes and 6 3D thick modes), mip chains/tails, arrays, cubemaps, and volume textures.
* **Ray Tracing & Upscaling:** Runs console-native `XboxBvhBuilder` acceleration structure builds and AMD FSR 4 console DLL passes (`amd_fidelityfx_upscaler_xbox.dll`) directly on PC GPUs.
* **Audio & Video:** Routes XAudio2 to Windows `XAudio2.9` (with Opus via `MSOpusDecoder.dll` and pure-Rust XMA1/XMA2 decoding) and supports console Bink 2 video playback.

---

## Controls

A **gamepad is strongly recommended**, as console titles expect controller navigation.

* **Controller:** Any XInput-compatible gamepad (Xbox wired/wireless) works out of the box with full stick, trigger, button, and rumble support (hot-pluggable).
* **Keyboard & Mouse Emulation:** Mapped automatically as a virtual gamepad (can be used alongside a controller or disabled via `keyboard_mouse = false` in `xcl.toml`):
  * **Movement / Camera:** `WASD` (Left Stick) · `Mouse` (Right Stick) · `Shift` / `Ctrl` (L3 / R3)
  * **Face Buttons:** `Enter` / `Space` (A) · `Esc` / `Backspace` (B) · `Q` (X) · `E` (Y)
  * **Shoulders / Triggers:** `Mouse Wheel` (LB / RB) · `Left / Right Click` (LT / RT)
  * **System:** `Arrow Keys` (D-Pad) · `Tab` (View) · `P` (Menu)

---

## Prerequisites & Building

### Requirements
* **OS:** Windows 10 (22H2 KB5029331+) or Windows 11 (x64)
* **Toolchain:** Rust (`stable-x86_64-pc-windows-msvc`)
* **Windows SDK & DXC:** `dxcompiler.dll` + `dxil.dll` (**DXC 1.8+** is strongly recommended for faster pipeline compilation and smaller DXIL output; download from [DirectXShaderCompiler releases](https://github.com/microsoft/DirectXShaderCompiler/releases) and set `XCL_DXC_DIR` to its `bin\x64` folder).
* **GPU:** DirectX 12 GPU supporting **Shader Model 6.6** and **Mesh Shaders**.
  * *Windows 10 Note:* Inbox `D3D12Core.dll` stops at SM 6.5. Place the Microsoft D3D12 Agility SDK (`D3D12Core.dll`) in a `D3D12\` subfolder next to `xcl.exe` or set `%XCL_D3D12_DIR%`.
  * *NVIDIA / Wave32 Note:* Console wave64 shaders execute as two 32-lane halves on Wave32 hardware (see [`docs/wave64-on-wave32.md`](docs/wave64-on-wave32.md)), which incurs a performance overhead compared to native Wave64 GPUs.

### Build from Source
```powershell
git clone https://github.com/itsF4LCON/Prodigy.git
cd Prodigy
cargo build --release --workspace
```
The compiled binary will be located at `target\release\xcl.exe`.

---

## Usage

Double-clicking `xcl.exe` from Windows Explorer launches a minimal GUI picker to select a game executable or `MicrosoftGame.config`. For advanced control and diagnostics, use the CLI:

```powershell
# Launch a game
xcl run D:\Games\MyTitle [-- optional game args]

# Inspect a game's executable imports and compatibility coverage
xcl inspect D:\Games\MyTitle\game.exe

# Disassemble raw RDNA2 shader binaries or console containers
xcl disasm <shader.bin> [--hex]

# Analyze or precompile a shader pack
xcl shaders D:\Games\MyTitle\data\shaders.rpf --top 50
xcl shaders D:\Games\MyTitle\data\shaders.rpf --precompile --only cs --pipelines

# Extract assets from an RPF archive
xcl rpf D:\Games\MyTitle\data\shaders.rpf --extract NAME --out output_folder
```

### Data & Logs
Per-title runtime data is stored in `%LOCALAPPDATA%\XCL\<title>\` (overridable via `%XCL_DATA_ROOT%`):
* `xcl.toml` – Per-game configuration (input settings, overrides)
* `overlay\` – Redirected file writes (keeps your game folder clean)
* `saves\`, `pls\`, `temp\` – Save data, persistent storage (`S:\`), and scratch space (`T:\`)
* `cache\` – Compiled DXIL shader and pipeline caches (`XCL_TRANSLATED_CACHE=0` to disable)
* `logs\report.json` – Diagnostic summary generated after every run (unimplemented stubs, shader stats, DXC version, crash info)

---

## Running Tests

```powershell
.\scripts\test.ps1
```
Runs the full workspace unit test suite, an end-to-end test title (`tests/testtitle`), and D3D12 GPU shader validation tests.
* Use `XCL_ADAPTER=<index|name>` to target a specific GPU.
* Set `XCL_REQUIRE_GPU=wave64,mesh` to fail instead of skipping tests when hardware features are missing.
* Retail sample tests (`XCL_SAMPLE_RPF`, `XCL_SAMPLE_DLLS`, `XCL_SAMPLE_TTPF`, etc.) are opt-in via environment variables pointing to your own dumps (see `HANDOFF.md`).

---

## License

This project is licensed under **GPL-2.0-only** — see [`LICENSE`](LICENSE) for details.  
Third-party acknowledgments and reference sources are documented in [`THIRD_PARTY.md`](THIRD_PARTY.md).