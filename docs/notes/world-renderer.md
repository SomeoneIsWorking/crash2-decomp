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
0x19, 0x20, 0x21; sway 0x0D, 0x23; glow 0x0C, 0x25, 0x27; the warp room 0x02 lamps on one run and tint on another (its zones). Fog
and light were not selected at any entry: fog's flag comes from zones further in, light's from the game's flag.
