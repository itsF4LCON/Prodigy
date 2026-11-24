# Contributing to Prodigy

Prodigy is a newly started project with no external contributions yet—help, bug reports, GPU test reports, and pull requests are all welcome.

## Getting Started

Prodigy builds and runs on **Windows 10/11 (x64)** with Rust (`stable-x86_64-pc-windows-msvc`).

```powershell
cargo build --workspace
.\scripts\test.ps1
```

If you are planning a larger change, open an issue first to discuss the approach before spending time implementing it.

## Pull Requests

- **Keep changes focused:** One logical change per PR makes review much easier. Avoid unrelated refactors or drive-by reformatting (`cargo fmt` on untouched files).
- **Describe what you tested:** Note your Windows version, GPU, and the output of `.\scripts\test.ps1` (including any `SKIPPED` GPU tests). Be honest about what you could not test.
- **Add tests for logic changes:** Put unit tests in `#[cfg(test)] mod tests` and integration tests in `crates/prodigy/tests/`. GPU tests should skip cleanly via `gpu::recomp::runner::skip_test(...)` when required hardware features are missing.
- **Update `README.md`** if your change affects user-facing behavior or game compatibility.

## AI-Assisted Contributions

AI coding tools are welcome. Please review and understand the diff yourself, actually run the build/tests before opening a PR, and briefly mention which tool/model helped in the PR description.

## Legal & License

Prodigy is licensed under **GPL-2.0-only** ([`LICENSE`](LICENSE)).

- **No copyrighted or SDK material:** Never commit game files, dumps, keys, `MicrosoftGame.config` from retail titles, console SDK headers/binaries, or DRM circumvention code.
- **Dependencies:** Any new crate must be GPL-2.0-compatible (e.g. MIT, BSD, Zlib, ISC, Unlicense, LGPL-2.1+, GPL-2.0).
- **Attribution:** Code adapted from another compatible open-source project should note its origin and be listed in [`THIRD_PARTY.md`](THIRD_PARTY.md).
