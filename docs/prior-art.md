# Prior art — Crash Bandicoot 2: Cortex Strikes Back (`SCUS_941.54`)

Outside work on `SCUS_941.54` (SHA-1 `aabdade44a4ca863f71224daef51c45f34fb1bf8`); a reference names addresses, our measurement decides behaviour.
Address checks ran psxport `decomp_pipeline.py --function-at` on our image (2026-10-13) unless marked "our docs";
licences, trees and push dates were re-read through the GitHub API on 2026-11-13. Paths such as `docs/issues/...` are
psx-crash paths.

Tools that cover all three titles (GOOL bytecode, NSF/NSD data) are listed in `crash-bandicoot-decomp`'s
`docs/prior-art.md`.

## Decomp

| submodule | URL | pinned | licence | target build and match | progress (verified in tree) | what we may take |
|---|---|---|---|---|---|---|
| `external/crash2-decomp` (psx-crash) | https://github.com/Gabriel-TwoBits/crash2-decomp, forked as https://github.com/SomeoneIsWorking/crash2-decomp (this repository) | `d7da30d0` (2026-10-25) | GPL-3.0 | Crash 2 `SCUS_941.54`: `config/SCUS_941.54.sha1` is our SHA-1; splat config `config/SCUS_941.54.yaml` | README: 270 of 466 game functions byte-matching (20.53% of bytes), the rest behaviour-equivalent C tested against retail; 33 `src/**/*.c`, per-function notes in `config/match/notes/` | evidence only (GPL): names, types, function semantics; no code copied into our MIT tree |

## Other projects

| project | URL | kind | licence | target build | build match (addresses checked) | what we may take | notes |
|---|---|---|---|---|---|---|---|
| ignzio/CB2-Online | https://github.com/ignzio/CB2-Online | PC runner (own R3000/GTE/GPU/SPU platform layer running the retail code) with online play | MIT (`LICENSE` text; GitHub reports NOASSERTION) | Crash 2 `SCUS_941.54` and PAL `SCES_009.67` | not checked: no address list read | may copy with attribution | Its 60 fps (`src/interp.cpp`, `interp.hpp`, read 2026-11-13) is screen-space: each triangle of the newest GPU command list is matched to one in the frame before by texture, texel and colour key, then moved halfway; nothing reads game state, so it is image-space motion estimation and nothing for a replay port to take. GOOL VM and NSF/NSD readers in `src/gool_*.cpp`, `src/nsf.cpp`, `src/nsd.cpp`. Last push 2026-11-05. |
| Gabriel-TwoBits/crash2-pc | https://github.com/Gabriel-TwoBits/crash2-pc | native PC port compiling the crash2-decomp C (submodule `decomp/`) | GPL-3.0 | Crash 2 `SCUS-94154` via the decomp | matched through its submodule | evidence only | 60 fps in `src/between.c` (read 2026-11-13) over the decomp's `FRAME_BETWEEN` hooks (`include/platform.h`): at a game frame's end it runs `Sys_MainLoop` once more with none of the logic (no pad, no GOOL, no `Cam_Update`, no clocks), with the camera's six words from `g_CameraX` 0x800607F4 and each object's nine pose words at +0x60 (position, angles, scale) set halfway, then puts them back; poses step at 30 (no animation blend). No widescreen code. Last push 2026-10-25. |
| ughman/c2c | https://github.com/ughman/c2c | MIPS-to-C translation plus hand decomp | none | Crash 2 PAL `SCES_009.67` | mismatched: `CORE_VSync 0x8004A864` vs our `0x8004A484`; pre-shift code agrees (`CORE_Main 0x800117BC`, `CORE_Loop 0x80011800`) | evidence only (names, after measuring the PAL offset) | Last push 2020-09-30. |
| Zumbo06/Crash2Recomp | https://github.com/Zumbo06/Crash2Recomp | psxrecomp static recompilation | none on GitHub; README states PolyForm Noncommercial 1.0.0 for psxrecomp | Crash 2 `SCUS-94154` | matched: seeds contain our `0x800117BC`, `0x80049BD4`, `0x80015614`, `0x8004B1B8`, `0x8004EC30`, `0x80049D1C` | nothing from the runtime; the seed list is a function-entry census | Last push 2026-10-08. |
| Purplesmaug96/c2-recomp | https://github.com/Purplesmaug96/c2-recomp | psxrecomp static recompilation | MIT | Crash 2 `SCUS-94154` | matched: `game.toml` entry `0x80049B2C` | nothing from the runtime | Single push 2026-08-01. |
| Matteo842/CrashBandicoot2-Launcher | https://github.com/Matteo842/CrashBandicoot2-Launcher | static recompilation (RecompOne) plus launcher | MIT | Crash 2 PAL `SCES_009.67` | mismatched build: PAL | nothing | Not playable per its README. Last push 2026-09-16. |

Not found (2026-11-13): any Crash 2 entry on decomp.dev. `dguillot-gh/crash2-android` is gone (404).
