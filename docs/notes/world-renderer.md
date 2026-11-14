# World renderer — Crash Bandicoot 2 (`SCUS_941.54`)

`Gfx_DrawWorlds` (0x80041E5C) and every step it jumps through, read from the retail disassembly. It is
handwritten assembly that shares registers across steps, so it is described here rather than matched; the C in
`src/game/80039D2C.c` is its non-matching equivalent. `docs/TARGET.md` ("The world renderer") has the overview and the
mode table; `screen-culls.md` the screen test.

## Dispatcher

- Prologue saves s0-s7, t8, t9, gp, fp, ra in a 0x44-byte frame and stores that `$sp` at scratchpad 0. TRX, TRY and
  TRZ are zeroed, the clock goes to 0x1C, and `$sp` becomes the sort list's last entry (list + 2 + 2 × count, the
  count a signed halfword); list + 4 is the first entry, kept at 0x9C. When they are equal the list is empty:
  `Gfx_ResetOt(frame + 0x18, 0x800)` and return.
- Otherwise: the camera's position >> 8 to 0xC0..0xC8; the scene's projectors and drawers (scene + 0x14..0x20) to
  0xA8, 0xAC, 0x64, 0x68; the setup (scene + 0x10) is called with `$v1` = 0x1F800000 and `$a2` = scene (every setup is
  a leaf reading only those); the depth-shift table (8 words at 0x8005C724) to 0xD0; 0xFFF0FFF0 to 0x1D8, 0xFFFEFFFF to
  0x1DC, the bound 0x00D90200 to 0x21C; ordering table slot 0x7FF gets the end marker 0x00FFFFFF and is the lowest
  slot linked so far.
- The loop decodes one entry ahead: each projector reads the next entry right after RTPT. Polygons that are culled
  or skipped return to 0x80042390, linked ones through the link tail at 0x80042948; both reload the colour table from
  the next entry's world and go back to the loop top (0x8004204C) until `$sp` reaches the first entry.
- The entry at the first slot has been decoded but not drawn when `$sp` reaches it. Scratchpad 0x6C, zeroed before
  the loop, is set to 0x1F800000 on the first arrival and the loop runs once more for that entry; the second arrival
  finishes. Then the slots below the lowest one linked are chained down to slot 0, the primitive cursor goes back to
  frame + 8, and the saved registers come back.

## Loading a polygon

A vertex's VXY is `((word & 0xFFF0FFF0) + world_xy) & 0xFFFEFFFF`, its VZ `(half & 0xFFF0) + world_z` (32 bits).
Corners by kind: triangle (first >> 20, first >> 8, second >> 4, 12 bits each), quad-first (first >> 20, first >> 8,
second >> 20), quad-last (first >> 8, second >> 20, second >> 8; 0x184 = 0xFFF0FFF0 turns its texture), quad (all four;
the fourth stays in registers, not the GTE). The three-corner kinds store 0x184 = 0 or the turn mask; the quad leaves
0x184 alone. The default quad projector stores SXY0 at the next primitive + 8 before it culls, so that word changes
for culled quads too.

## Linking

The link tail relinks downward when the polygon's slot is below the lowest linked: each slot between points to the one
above, the polygon goes to the head of the new lowest slot. A polygon whose slot is above the lowest goes into the
lowest slot, not its own: the list is far first, so that only happens within one slot's depth.

## Modes

- fog: IR0 from 0x90 (scene + 0x3C); each corner DPCS toward 0xFF0 (bit 18) or 0x200.
- depth fog: corners deeper than the start (scratchpad 0x10) are DPCS'd with IR0 = (depth − start) << the shift at
  0x08 (both from the setup); when the last corner was
  fogged and every colour is 0, an F3 or F4 in black instead (5 or 6 words).
- tint: IR0 from 0x214; bit 19's corners DPCS'd (the far colour set by the setup).
- pulse: any bit 19 corner pulses (IR0 from 0x84, the setup's triangle wave of the clock); else, with bit 18 in the
  first vertex, every corner dims to black from depth 0x258 over 0x2D8, and a polygon left all black is skipped.
- lamps: the quad projector 0x80042698 keeps VXY0/VZ0 at 0x1B4/0x1B8 first. Per corner (0x80042D10): bit 19 tints,
  bit 18 keeps its colour, otherwise the two lamps' reach (0x1320 less |dz|, |dy|, then 0x320 and |dx|) sums to a
  light that picks the far colour, the full colour, or a DPCS between, then DPCL with IR0 = 0.
- glow: the first corner decides which lamps reach the polygon (reach 0x15E0); the later corners take the lamps that
  reached (reach 0x12C0); bit 19 adds 0x4CB. Far from both lamps and with no bit 18 or 19 vertex, a black F3/F4.
- waves: bit 18's vertices get `waves[((x + y + phase) >> 4) & 15] << 16` (phase at 0xCC is camera x + y); polygons
  whose vertices all have bit 19 dim from 0x258 over 0x3A0.
- sway: bit 18's vertices move by the sine table at 0x8005BCB0 (helper 0x800429D0, angle from x + z + phase and the
  clock >> 0xF0); bit 19's corners take the colour at 0x1A8 when it is nonzero.
- light: see below.

## Light

The light mode's projectors (0x80045D88, 0x80045F40) fall back to the default ones unless some vertex has bit 18,
some corner's z is within 0x400 of the light's (0x98) and some corner's x within 0x400 of the light's (0x94). A polygon
that passes is culled like the default, then walked (0x800461E0) as one triangle, or as two for a quad (corners 1, 2, 3
with the texture turned first, then 0, 1, 2):

- The triangle is divided into 45 points, 9 rows of 9 down to 1, stepping x, y, z, colour, the texture's u and v and
  the light-map cell by an eighth of each edge (scratchpad 0x110..0x16A). Red and u steps lose bit 15; green, blue and
  v keep it; packed pairs step with bit 15 of the low half cleared.
- At each point within the 16 by 16 maps (0x1CC shades, 0x170 heights): the colour is DPCL'd with IR = shade × 32; a
  height of 0xFF leaves the point unlit (its colour word −1); otherwise the point is raised by the height, projected,
  and its colour, SXY, SZ and uv kept in the buffer at 0x1D4 (12 bytes a point).
- The strip list at 0x8005C6D0 (a set top bit starts a strip, −1 ends) draws each new point with the two before it,
  as G3 or GT3, linked at its own depth less 5 slots (into its own slot when that is above the lowest linked) and
  skipped when a corner is unlit.
- Then the whole triangle is drawn 40 slots farther back with its own colours and texture, through the ordinary link.

## Reached by warp

At each level's entry (`warp scene`, 6-8 s): default in most scenes; pulse 0x12, 0x1A; depth fog 0x07, 0x16; waves
0x19, 0x20, 0x21; sway 0x0D, 0x23; glow 0x0C, 0x25, 0x27; the warp room 0x02 lamps on one run and tint on another (its
zones). Fog and light are not selected at any entry.

## Where fog is selected

Fog needs the scene flag 0x200000 (property 0x185 of the camera path at its progress) and `D_8006CDC4`, GOOL global 149,
nonzero; with the flag and global 149 zero the scene takes depth fog. Read from every zone (type 7) of every NSF on the
disc, only level 0xF carries 0x200000: its `_fZ` zones 02 (from progress 23) to 17 and B2 to B5. The executable only
reads global 149 (0x8002155C); its writer is GOOL `LighC` (levels 0xF and 0x1E), the lightning: `GVAW` 149 with a
falling value per flash, then 0. Level 0x1E's zones have no 0x200000, so its lightning never selects fog.

Level 0xF's spawn point 1 (zone `05_fZ`, the camera in `04_fZ`, flags 0x200090) is one the game starts the level at
(`spawn-index.md`). From it, with no input, the scene switches between depth fog and fog with each flash (30 s: 32 of
120 polls in fog).

## Where light is selected

Light needs bit 0x1000000 of `g_FrameFlags`, which `Gfx_EndFrame` copies from GOOL global 4. No C stores that bit
(the level start stores 0x0C01FFFF). GOOL `WilnC`, the player's code for levels 0x15 and 0x24 (no pool of its own: it
runs on `WillC`'s, whose word 0x7A is 0x1000000), ORs it into global 4 in `WillC`'s states 0x74 to 0x7A (state
descriptors naming external entry 7, `WilnC`):

- state 1's trans hook (`WillC` 0x315) goes to state 0x74 after a pad test (`CPAD` at 0x322) when global 0 is level
  0x15 or 0x24 and the player's `status_a` holds all of 0x4000001 (`TST` is (A & B) == A); state 0x74's code (`WilnC`
  0) ORs the bit and sets global 144 (`D_8006CDB0`, which `Gfx_SetupScene` hands
  `func_8002FC78` shifted by 8) to 0x700;
- state 0x3F (events 0xE00, 0x1900, 0x2500 through item 3) goes to state 0x79 in those levels, whose code (`WilnC`
  0x181) ORs the bit only when `status_a` has 0x4000000.

In the C and in level 0x15's GOOL, `status_a` 0x4000000 is stored only by `func_8002A1B8` case 15, the callback `func_8002A3B0` hands `Zone_FindFloorY`
(0x8002A524): the player stands on floor of type 15. A scan of the zones' octrees (each halfword from the root on with
bit 0 set read as a leaf, its type in bits 4 to 9) finds type 15 only in levels 0x15 and 0x24, in level 0x15's zones
`1l` to `1v` and in 27 of level 0x24's. Neither level's spawn points 0 and 1 stand on it (`status_a` 0x00060801
there), so no warp the game itself makes reaches light.

With global 4's bit and global 144 (0x700) set through the debug server at both levels' spawn points, the scene takes
the light mode, but no polygon there passes the projectors' test: in 6 s at level 0x15's spawn 1 no store reached the
walk's scratchpad (0x110 to 0x16A) during `Gfx_DrawWorlds`, so the walk (0x800461E0) is still not observed running.
