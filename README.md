# Monsters, Inc. Scare Island Recompiled

<!-- retcomm-readme-metrics -->
[![GitHub downloads (all assets, all releases)](https://img.shields.io/github/downloads/TechnicallyComputers/ScareislandRecomp/total)](https://github.com/TechnicallyComputers/ScareislandRecomp/releases)
[![GitHub downloads (latest release)](https://img.shields.io/github/downloads/TechnicallyComputers/ScareislandRecomp/latest/total)](https://github.com/TechnicallyComputers/ScareislandRecomp/releases/latest)
[![GitHub release](https://img.shields.io/github/v/release/TechnicallyComputers/ScareislandRecomp)](https://github.com/TechnicallyComputers/ScareislandRecomp/releases/latest)
<!-- /retcomm-readme-metrics -->

Static recompilation of **Disney/Pixar Monsters, Inc. Scare Island** (PS1, PAL) built on
[psxrecomp](https://github.com/mstan/psxrecomp) and
[recomp-ui](https://github.com/mstan/recomp-ui).

The game's MIPS code is translated ahead of time into C and compiled into a
native Linux / Windows / macOS executable. You supply your own disc; nothing
copyrighted is in this repo.

| | |
|---|---|
| Players | 1 |
| Region | Europe (PAL) |
| Target disc | SCES-03759 (English), boot EXE `SCES_037.59` |
| Other PAL releases | SCES-03760 … 03769 (Danish, Dutch, German, Spanish, …) work too; `scripts/build_from_disc.sh` re-wires the boot-EXE name automatically |
| US release | *Monsters, Inc. Scream Team* (SCUS-94635), **tested**: boot, FMVs, menus, character select, tutorial gameplay (walk / jump / dialog), save states, zero missed functions |
| Publisher | Sony Computer Entertainment |
| Developer | Artificial Mind & Movement (A2M) |
| Year | 2002 |

Scaffolded with the New Project Layout. See
`psxrecomp/docs/GAME_PROJECT_SETUP.md` for the full flow.

<!-- retcomm-readme-launcher -->
## RetComM Launcher

You can run this title **standalone** (release zip + the built-in recomp-ui
Generate & Build flow), or manage installs, updates, ROM/BIOS wiring, and queued
builds more intuitively with
**[RetComM Launcher](https://github.com/TechnicallyComputers/RetComM-Launcher)** —
the Retro Compilation Manager hub for self-compiling recomps.

[Downloads](https://github.com/TechnicallyComputers/RetComM-Launcher/releases) ·
[Full README & features](https://github.com/TechnicallyComputers/RetComM-Launcher#readme)

<p align="center">
  <img src="https://raw.githubusercontent.com/TechnicallyComputers/RetComM-Launcher/main/docs/screenshots/hub-and-game-launcher.png" alt="RetComM hub with a background build, next to a title’s recomp-ui launcher" width="720">
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/TechnicallyComputers/RetComM-Launcher/main/docs/screenshots/queue-and-background-build.png" alt="Background cmake build with titles queued" width="720">
</p>

RetComM checks for updates, rebuilds with existing build data when possible,
shares the portable toolchain used by per-title launchers, and automates
BIOS/ROM/save plumbing so you are not stuck repeating each game’s wizard by hand.
<!-- /retcomm-readme-launcher -->

## Legal

You must own the original game. Disc images under `disc/` are gitignored and
must never be committed. Retail BIOS dumps are not redistributed; OpenBIOS is
used for Generate unless you supply your own SCPH locally.

Default app icon: `assets/psxrecomp.ico` (and `.png` / `.svg`) — RetComM-themed controller mark from `psxrecomp/assets/`. Windows builds embed it via `APP_ICON`.

Optional box art under `launcher_assets/img/` may come from
[libretro-thumbnails](https://github.com/libretro-thumbnails/libretro-thumbnails)
(`Named_Boxarts`); see `BOXART_SOURCE.txt` when present.

## Quick start (Linux)

You need a **Redump-style BIN/CUE dump of your own disc**. ISO-only rips are
accepted but get normalized; a full multi-track cue is best.

```bash
# 1. Clone WITH submodules. GitHub's "Download ZIP" leaves psxrecomp/ empty.
git clone --recursive https://github.com/jns630/Monster-s-Inc-Scare-Island-Recompiled-PS1
cd Monster-s-Inc-Scare-Island-Recompiled-PS1

# 2. Build dependencies (Ubuntu / Debian)
sudo apt install build-essential git python3 cmake ninja-build pkg-config \
  libgl1-mesa-dev libx11-dev libxext-dev libxrandr-dev libxcursor-dev libxi-dev \
  libxss-dev libxtst-dev libxkbcommon-dev libwayland-dev wayland-protocols \
  libegl1-mesa-dev libasound2-dev libpulse-dev libudev-dev libdbus-1-dev

# 3. Disc -> recompiled C -> native exe, in one go
scripts/build_from_disc.sh "/path/to/Monsters, Inc. - Scare Island (Europe).cue"

# 4. Play
./build-release/Scareisland_Recompiled
```

`build_from_disc.sh` builds the recompiler, probes the disc (writing
`game.toml`, `seeds/ghidra_funcs.txt` and `catalog_identity.json`), keeps the
boot-EXE name in `CMakeLists.txt` / `codegen_setup.c` in sync with your disc,
runs Generate and builds `psx-runtime`. Options: `--bios SCPH1001.BIN` to use a
retail BIOS dump instead of OpenBIOS, `--debug` for a build with the TCP debug
server (port 4370) on, and `-- -DFOO=BAR` to pass extra CMake flags.

The probe writes your disc's **absolute path** into `game.toml`, so don't
commit `game.toml` after running it. `git checkout game.toml` restores the
baseline.

Manual equivalent:

```bash
./psxrecomp/tools/ci/build_emitters.sh
python3 psxrecomp/tools/new_project_layout/probe_disc.py disc.cue \
  --write-game-toml game.toml --write-seeds seeds/ghidra_funcs.txt \
  --write-catalog catalog_identity.json --disc-rel "$PWD/disc.cue" \
  --out-dir disc --write-boot-exe disc --players 1
python3 psxrecomp/psxrecomp_cli.py generate --config game.toml --project-root . --disc disc.cue
cmake -S . -B build-release -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build-release --target psx-runtime
```

Zip prefix for CI artifacts: `scareisland`.

## Enhancements

All of these were tested on the US disc (SCUS-94635) at 1920×1080.

| Feature | Where | Default | Notes |
|---|---|---|---|
| **Upscaling** | `game.enhancements.toml` → `[video] supersampling` | **4×** (1280×960 internal) | 1–4. Clean polygon edges, no artifacts seen. |
| Texture filtering | `[video] texture_filtering` | `nearest` | `"bilinear"` smooths the texels. |
| **PGXP precision** | `build-release/mods/state.toml` | **on** | Stops 3D wobble and fixes texture warping. Needs the framework patch in `patches/psxrecomp/` (applied by the build script). |
| **Fast Loading** | `mods/state.toml` | **on**, 4× | Host-side only; the game can't tell. |
| CD Speed | `mods/state.toml` (`psx.enhancement.cd-speed`) | off | Guest-visible alternative to Fast Loading. |
| **Widescreen** 16:9 / 21:9 | `mods/state.toml` (`monstersinc.enhancements` / `widescreen`) | off (experimental) | Real wider 3D view in levels; menus, loading cards and FMVs stay 4:3. Things the game culls at the old 4:3 edge leave dark gaps at the far sides, worse at 21:9. |
| CRT / scanlines | `[video] crt_filter`, `scanlines` | off | `"crt"`, `"composite"`, `"trinitron"`. Framework feature, not tested on this game. |

Edit `game.enhancements.toml`, not `game.toml`: the disc probe rewrites
`game.toml` and the build script merges these sections back afterwards.
`mods/state.default.toml` is copied to `build-release/mods/state.toml` on the
first build; after that the file is yours.

This repo ships no recomp-ui launcher, so there's no in-game Mods page. The
toml files above are the controls.

## Symbols

Progressive map: `symbols.toml` → `python3 tools/sync_symbols.py` →
`psx_symbols.h` (`PSX_FN_*`). See `psxrecomp/docs/SYMBOLS.md`.

## Framework pins

Submodule gitlinks (`psxrecomp`, optional `recomp-ui`, nested `recomp-net`)
are authoritative. `framework_pins.txt` is an optional scaffold snapshot;
release CI logs SHAs with `record_pins.sh` but builds whatever the gitlinks
resolve to. Bump submodules deliberately — do not float on `main`/`master`
in release CI.

<!-- retcomm-readme-raid -->
---

<p align="center">
  <sub><b>R.A.I.D. — Retro AI Development</b> · a Discord for AI-assisted retro reverse-engineering, decomp &amp; recomp</sub>
</p>

<p align="center">
  <a href="https://discord.gg/Ad9BwSzctP"><img src=".github/raid-discord.png" alt="Join the Retro AI Development (R.A.I.D.) Discord" width="200"></a>
</p>
<!-- /retcomm-readme-raid -->
