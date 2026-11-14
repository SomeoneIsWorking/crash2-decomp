# Screen-edge culls — Crash Bandicoot 2 (`SCUS_941.54`)

Read from the retail executable's disassembly. The screen the culls test is 512 by 216: the packed word
0x00D90200 (217 << 16 | 512) and the immediates 0x200 and 0xD8.

## World polygons

`Gfx_DrawWorlds` (0x80041E5C) stores 0x00D90200 at scratchpad 0x1F80021C (`lui`/`ori` at 0x80041FA0/0x80041FA4,
`sw` at 0x80041FA8) after the scene mode's setup (`jalr` at 0x80041F34) has run. Its sort-list loop reaches each
polygon's projector by `jr $t9` through scratchpad 0xA8 (triangles) or 0xAC (quads), copied from `g_Scene` + 0x14 and
+ 0x18; the projector reaches the draw step by `jr $a2` through 0x64 and 0x68. None of these steps is called: they
are jumped to with `$ra` holding data.

The default mode's projectors (`Gfx_SetDefaultSceneMode` stores them) are `func_80042420` (triangle) and
`func_800426A0` (quad). After RTPT each loads the bound into `$ra` (0x800424B4, 0x80042748) and rejects the polygon,
branching to the loop's next entry (0x80042390), when every corner is past the same edge:

- both halves at once: `nor((s0 - b) | (s1 - b) | (s2 - b))` has the lane's sign set when each corner's x (low
  half) or y (high half) is at least the bound's; the low half's borrow reaches the high half;
- `s0 & s1 & s2` has the lane's sign set when each corner's x or y is negative;
- the two are ORed; bit 31 (y) is tested, then bit 15 (x) after a shift by 16.

So a polygon is drawn when some corner has 0 <= x < 512 and some corner has 0 <= y < 217. The left and top edges
are the sign bits of the GTE's SXY words; only the right and bottom edges come from the bound. The bound has two
more readers, 0x80045E98 and 0x80046078, in the other scene modes' projectors.

The zone's sort list carries polygons beyond the 4:3 right edge: in scene 0x15 at 16:9 (684-wide record canvas),
with the immediate at 0x80041FA4 written to 0x3FF through the debug server, the right margin fills with walls
and floor the retail bound rejected, and the 4:3 picture is unchanged.

The list itself is disc data for the 4:3 view: `func_80029800` copies a whole SLST list or applies a delta to the
previous one (`Gfx_DecodeSortListForward`, `Gfx_DecodeSortListBackward`), with no screen test. In the warp room (scene
0x02) at 16:9 the list's polygons reach SX -376 to 664, but none covers the margin past the Snow Go doorway (SX 560 to
595 at SY 30, 580 to 595 at SY 90): that gap is missing from the list, not culled.

## Object box test

`Gfx_QueueObject` stores a packet's visibility test at packet + 0x1C: `func_80041E50` (always visible, `t8` 0)
when the object's word 0xAC has bit 0x40000 set, otherwise 0x80041D14. The object drawers call it with
`jalr $t8` and skip the model when it returns `t8` nonzero (0x8003DF14, 0x8003DF1C).

0x80041D14 saves `$ra` at scratchpad 0x64, reads the frame's box corner from scratchpad 0x118 (packed x, y) and
0x11C (z), and projects the box's corners three at a time (the extent 0x3FC on each axis, the x sum masked by
`$t2`): (x, y, z), (x+e, y, z), (x+e, y+e, z); (x, y+e, z), (x, y+e, z+e), (x, y, z+e); then VXY1 and VXY2 become
(x+e, y) and (x+e, y+e) with VZ1 and VZ2 still z+e. Each projected corner (SXY and the matching SZ) goes through the
subroutine at 0x80041E20, entered by `jal`, which returns to the next corner when the corner is off screen and
otherwise returns from the whole test with `t8` 0 through the `$ra` saved at 0x64. A corner is on screen when
x >= 0, y >= 0, y < 0xD8 and x < 0x200 (x and y the SXY halves); after all eight, `t8` is -1. Both exits restore
`$ra` from 0x64.

## Object triangles

An object routine at 0x80041BE8 builds the same 0x00D90200 inline (0x80041BF0/0x80041BF4) and applies the world
projectors' all-corners-past-an-edge test to a triangle's three SXY words.
