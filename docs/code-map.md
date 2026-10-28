# ratchet-clank-pc code map

- **Repo:** iannitram/ratchet-clank-pc (public fork). **Kind:** C loader and runtime plus a C++ GPU library (CMake and Ninja, driven by `build.sh`), Python offline tools and a GTK4 launcher: a native Linux port of *Ratchet & Clank* (2016, PS4), built on the Bloodborne port (bbport).
- **Upstream:** btsouth/ratchet-clank-pc (`upstream/main`). **Integration branch:** `main`. This is the kept copy of a pair; `ratchet-clank` is the superseded one.
- **Describes commit:** `2c6683a` (2026-10-27).

The game's original x86-64 executable runs directly on the CPU, a small runtime stands in for the PS4 system libraries, and a Vulkan renderer derived from shadPS4 draws the frame, with FSR and DLSS upscaling. It keeps the Bloodborne port's loader, runtime and renderer and adds a Ratchet game profile, launcher and save tool. No game files are in the repo. Parked reference mirror: upstream owns all code.

## Layout

| Path | What it owns |
|---|---|
| `src/` | Loader (`probe.c`) and HLE runtime, one file per area (`runtime_kernel.c`, `_memory`, `_thread`, `_fiber`, `_evf`, `_psem`, `_file`, `_audio`, `_ajm`, `_pad`, `_savedata`, `_net`, `_udp`, and others); `import_names*.inc`; `vulkan_smoke.c` |
| `scripts/` | Offline preparation (`prepare.py`, `link_libc.py`, `link_modules.py`, `patches.py`, `mods.py`, `game_check.py`, `content_profile.py`, `online.py`, `respell.py`) and `ratchet_save.py` (shows a save, can write an unlocked copy) |
| `gpu/` | The renderer library `libbbgpu.so`; see the next table |
| `launcher/` | GTK4 launcher: the Ratchet one (`ratchet-launcher.sh`, `ratchet_launcher.py`, `ratchet_settings.py`) and the inherited Bloodborne one (`bb-launcher.sh`, `bbport_*.py`), plus `shell.nix` |
| `packaging/` | `appimage.sh`, `AppRun`, `relocate.py` (AppImage that runs without user namespaces), `bbport-entry.c`, `default.nix` |
| `patches/` | `Bloodborne.xml`, inherited community patches (the Ratchet frame-rate patches are in the code) |
| `tools/` | Developer tools: scripted runs (`ab.sh`, `bench.sh`, `soak.sh`), FSR 4 shader rewrites, `fsr4cap/`, `bridge_helper/`, `gen_runtime_nids.py`, `re/` (Python analysis helpers) |
| `tests/` | About 45 files: C, C++ and Python tests (runtime, pad, online, optional services, CP decoder, deferred pipeline, flip watchdog, fill-clear, scissor, shader metadata, upscaler pieces, Ratchet launcher and save, AppImage relocation) |
| `docs/` | Design notes (`upscaler.md`, `parallel_gpu.md`, `motion_vectors.md`, `ratchet-motion-blur.md`, `MODS.md`), release `CHANGES_*.md`; most of the rest (including `ROADMAP.md`, `bloodborne/`, `history/`) is inherited from the Bloodborne port, partly in Russian |
| `third_party/LibAtrac9` | Submodule: ATRAC9 decoder |
| `build.sh`, `run.sh`, `shell.nix` | Build driver (also `--test`), run wrapper (recognises the game by title ID), Nix dev shell |
| `README.md`, `LICENSE` | Project readme, license |
| `AGENTS.fork.md`, `CHANGELOG.fork.md`, `PIPELINE.md` | Our rules for this mirror, change history, stage record |

### Inside `gpu/`

| Path | What it owns |
|---|---|
| `shadps4/` | Vendored shadPS4 subset (`VENDOR.txt`: commit `b4e7ae7`, GPL-2.0-or-later, local changes marked `bbport:`, plus a few recompiler fixes backported): `video_core` (AMDGPU, buffer and texture caches, Vulkan renderer), `shader_recompiler`, `common`, `core` (selected libraries: gnmdriver, videoout, kernel pieces, avplayer) |
| `shim/` | Glue between runtime and renderer: `bbgpu.cpp`, game menu, GNM hooks, acquire slots, copy and free checks, window and ImGui |
| `cp/`, `layer/` | PM4 decoder; the memory layer of the new memory model |
| `games/` | Per-game profiles: `ratchet.cpp` (shaders and images read from frame captures of the EU release) and `bloodborne.cpp`, with `game_profile.*` |
| `bbnet/` | Online module with shadPS4 network code |
| `dlss_bridge/`, `tools/`, `patches/` | DLSS bridge library; an FSR 4 benchmark; patches applied to the FSR-Vulkan submodule (`0001`: FSR 4 profiling switches) |
| `third_party/` | `sirit`, `gcn`, `half`, `httplib`, `imgui`, `fonts`, `fsr-vulkan` (submodule) |

## Where our changes live

Upstream owns everything. `git log upstream/main..HEAD` is one commit, three files (`AGENTS.fork.md`, `CHANGELOG.fork.md`, `PIPELINE.md`). Upstream also keeps many topic branches (for example `rac-*` and `core-l1-pipelines`) that this fork does not carry.

## Build and test

```sh
git clone --recursive <repo> && cd ratchet-clank-pc
bash build.sh                          # out/ probe and out/gpu/libbbgpu.so
bash build.sh --test                   # ninja builds fill-clear, shader-meta and flip-watchdog tests and runs them
python3 -m unittest discover -s tests
BB_GAME_DIR=/path/to/CUSA01073 bash run.sh
```

- Needs GCC, CMake, Ninja, Python 3, glslang, SDL3, Vulkan headers and the libraries in `shell.nix`; the launcher needs PyGObject, GTK 4 and libadwaita. Submodules: `gpu/third_party/fsr-vulkan`, `gpu/third_party/imgui`, `third_party/LibAtrac9`.
- The repo has no CI workflow.

## Mac / PC / Linux

- Linux x86-64 with a Vulkan 1.3 GPU only (PIPELINE.md: platforms linux). Upstream tested on one NVIDIA desktop and, for correctness only, an Intel iGPU; AMD is untested there.
- It does not build on macOS or Windows. A Mac or Windows session can read and edit files but cannot build or run anything here (R04).
