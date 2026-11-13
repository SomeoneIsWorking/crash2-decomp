# RE frontier — Crash Bandicoot 2: Cortex Strikes Back (`SCUS_941.54`)

The ordered evidence chain the psx-crash port follows toward faithful behaviour, run with re-harness
`re_frontier.py` from this repository's root (from the port: `RE_FRONTIER_ROADMAP=external/crash2-decomp/docs/re-frontier.md`).
`where` and evidence paths (`game/...`, `titles/...`, `tools/...`, `docs/issues/...`) are psx-crash paths. A dependency
on a step kept in another title's roadmap is named in `notes` as `Cross-title dep`, not in `deps`. The trilogy's
roadmaps are in `crash-bandicoot-decomp` (Crash 1, and SHARED-01, the cross-title ownership step), `crash2-decomp` and
`crash3-decomp`.

The execution-method migration is ordered in psx-crash `docs/migration.md`. Static-path observations below are frozen
evidence for the frontier that Lightrec must reproduce; they are not instructions to regenerate, build, or run that
path, and they do not make it a product fallback.

Statuses: ✅ `re-verified` · 🟡 `re-partial` · 🔬 `in-progress` · ⬜ `todo` · ⏸ blocked upstream ·
➖ `skip-by-design` · ⚠️ `hack`.

## Target identity

### CRASH2-01 — select and measure the Crash Bandicoot 2 executable
- status: re-verified
- deps:
- evidence: The real USA disc's `SYSTEM.CNF` selects `SCUS_941.54`. The extracted 327,680-byte PS-X EXE has SHA-1 `aabdade44a4ca863f71224daef51c45f34fb1bf8`, SHA-256 `6e5b2449310b1ed87915f53c38f296f2152bfd0b42eaca5ef86f74a3d07b6043`, entry `0x80049B2C`, text `[0x80010000,0x8005F800)`, gp `0`, and stack `0x801FFFF0`. Embedded strings identify the North American area, `BASCUS-94154`, and `Crash Bandicoot 2`. `python3 tools/verify_executable.py --title crash2 --check --exe scratch/bin/crash2/SCUS_941.54` matches 11/11, including the VSync body fingerprint.
- where: `titles/crash2/executable.json`, `tools/verify_executable.py`
- gap: This proves executable identity and header only; CRASH2-02 separately binds it to the supplied disc and validates the first executed boundary. It does not authenticate the whole disc or prove boot.
- notes: The measured `SCUS-94154` title codeword and every address are independent of Crash 1's `SCUS-94900` integration.

### CRASH2-02 — provision the selected disc and verify the first crt0 call
- status: re-verified
- deps: CRASH2-01
- evidence: The shared serial-aware provisioning path publishes only after SYSTEM.CNF boots SCUS_941.54 and the executable matches 11/11 facts. Its 10-case both-answer suite includes three-way title environment isolation and wrong-title refusal. Against clean pinned psxport bc8c8897, cached real SCUS_941.54 passes the executable fact check, oracle_spike 43/43, and crt0_extract/independent CPU agreement 6/6 at executed jal ordinal 1, step 81,725, target 0x8001144C: gp 0x8005F17C, a0 0x8006F1F4, sp 0x801FFFF8, and heap size 0x0018FE08.
- where: `tools/provision_title.py`, CMake `crash2_oracle_boot_check`, gitignored `scratch/bin/crash2/SCUS_941.54`
- gap: This proves only the first real crt0 call boundary. No later Crash 2 execution, BIOS continuation, hardware, native boot, or gameplay is claimed.
- notes: The 6/6 real-byte cross-check was reproduced at psxport bc8c8897. No disc was configured for a fresh extraction in that run; prior SYSTEM.CNF provisioning evidence remains the disc-selection authority.

### CRASH2-03 — own measured executable facts through the derived runtime
- status: re-verified
- deps: CRASH2-02
- evidence: Against clean pinned psxport bc8c8897, Crash2Runtime directly inherits title-agnostic BoundaryRuntime, which inherits GameRuntime; legacy views and unmeasured runtime products remain null, and the title explicitly reports `guestVramIsPicture=false` because no frame producer exists. GuestProgramImage matches 15/15 fields from real SCUS_941.54 and shipping crt0_extract, including game main 0x80049BD4 independently observed at oracle call three; an altered global pointer produces one named disagreement, authoritative Clang CTest passes 5/5, and psxport_sync checks the same scratch/build-clang provenance.
- where: `titles/crash2/entry/facts.cpp`, `titles/crash2/entry/runtime.*` over `game/runtime/runtime.*`; `tests/runtime.cpp`
- gap: The typed runtime owns measured executable facts and game main `0x80049BD4`; CRASH2-04 preserves the later boundary evidence, and CRASH2-JIT-01 owns future product execution.
- notes: The refusing `BoundaryRuntime` is deleted; Crash 2 runs on `crash::Runtime` (CRASH2-JIT-01).

### CRASH2-04 — recorded resident boundary evidence
- status: re-partial
- deps: CRASH2-03
- evidence: Real SCUS_941.54 emits 998 candidates from 270 static seeds with zero overlays under clean psxport `8611d756` (emitter 2026-08-24.2). Generated execution and the independent Mednafen CPU agree 34/34 at each of eight calls: 0x8001144C, 0x800117BC, game main 0x80049BD4, 0x80015614, 0x8004B1B8, 0x8004EC30, 0x8004F1F8, and 0x80049D1C. Call eight is the measured addiu-a0-1/syscall-0 wrapper. At step 85898 the oracle records Cause `0x20` and EPC `0x80049D20`; the shipping HLE retains that record, returns prior IRQ 1, disables delivery, and both CPUs pop the Status mode stack once to resume at `0x80049D24`. Their first post-return state then agrees 34/34 at dynamic BIOS dispatch PC `0xB0`. The 16/16 suite includes altered-frontier, out-of-text seed, 7/8 short-window, repeated-target, altered-register, wrong-syscall-target, and wrong-selector refusals.
- where: frozen evidence only; the static translator, seeds, and boundary tests were deleted by the execution migration and are not restored. The typed runtime facts it bound live in `titles/crash2/executable.json` and `titles/crash2/entry/facts.cpp`.
- gap: Equality is proven from entry through the first post-syscall B0 dispatch boundary. The B0 HLE call itself, later BIOS/hardware boot, frame output, and the remaining 990 emitted bodies are unverified.
- notes: Static discovery counts are not execution provenance; only nine addresses are execution-proven. No Crash 1/3 seed or address was reused. Preserve this measurement without extending the static path; Crash 2 later reproduces it through Lightrec after Crash 1's gameplay gate.

## Preserved execution and ownership evidence

### CRASH2-VSYNC — identify and bound guest VSync
- status: re-partial
- deps: CRASH2-01
- evidence: Verified SCUS_941.54 contains canonical libetc VSync at `[0x8004A484,0x8004A5CC)`. The tracked body SHA-256 is `25097695f5ebd26728662b7989487b139e140ccf2129a14df10b30a003865b4e`. The executable gate retains altered entry/body controls, and the exact-pinned asset-free contract test proves the production platform owner returns a typed `FrameBoundary`.
- where: `titles/crash2/executable.json`, `game/frame/frame_driver.*`, `crash::Runtime::registerOverrides` (`StockVSync`)
- gap: Reached live through Lightrec at both GpuUpdate sites (CRASH2-FRAME).
- notes: The address is a Crash 2 fact; neither Crash 1 nor Crash 3 addresses are reusable.

### CRASH2-FRAME — recover one complete native-owned frame step
- status: re-verified
- deps: CRASH2-04, CRASH2-VSYNC
- evidence: CoreLoop `[0x80011800,0x80011E24)` resumes at `0x80011848`; GpuUpdate `[0x8001645C,0x80016A60)` waits at `0x80016838`/`0x80016868`; the empty stub `[0x80011E90,0x80011E98)` called at the loop end returns to `0x80011DC4` and ends the host frame. Runs to the hub and levels (CRASH2-JIT-01).
- where: `titles/crash2/executable.json` `runtime.native_frame`, `game/frame/frame_driver.*`
- gap: none for the frame step.
- notes: The stub boundary is Crash 2's; Crash 1 and 3 end at the GpuUpdate return.

## Runtime execution migration

### CRASH2-JIT-01 — migrate Crash 2 after Crash 1 is complete
- status: re-partial
- deps: CRASH2-02, CRASH2-03, CRASH2-VSYNC
- evidence: Boots to the title on the shared Crash owners; levels load and run through `warp`; the warp-room hub, its portal and the pause menu work headless after `warp 2`; New Game's scene 0x1C is not yet played through, issue 0029. Ghidra over `SCUS_941.54`. Scene request word `0x8005F3F0` (`-1` none, `-2` reload the saved scene at `0x80060988`), loaded id `0x8005F3EC`. CoreLoop `FUN_80011800` unloads through `FUN_8001521C` and loads through `FUN_80014C68`, which reads the 12-byte row at `0x80069034 + id * 12` (header sector, byte size, end sector) and has `FUN_800123F0` read the header with `CdRead`; empty rows (ids 0, 1, 4, 5, 0xB, 0x14, 0x2A to 0x3A) hang the load. Script op class 0xC sub 9 in `FUN_80037698` (0x80037DC8) writes `operand >> 8` to the request word, which a portal script can use (not observed live); death writes 0x3B (`0x80011A4C`). Live: warping 0x10, 0x15, 0x1B and 0x25 loads levels Crash runs in.
- where: `titles/crash2/` plus psxport executor
- gap: The hub portal's request is now observed live (forced Up from the hub spawn loads scene 0x1E), and the pause menu opens in the hub and in a level; the pause text formatter at `0x8001BAB0` calls BIOS A0:0x10 `atoi`. New Game requests scene 0x1C, which is not yet played through to the hub. Representative gameplay is not passed.
- notes: Do not inherit Crash 1 addresses, seeds, or behavior. Cross-title dep: CRASH1-JIT-02.
