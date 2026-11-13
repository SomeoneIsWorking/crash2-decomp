# Frame loop and draw section — Crash Bandicoot 2 (`SCUS_941.54`)

Read from the retail executable's disassembly of `Sys_MainLoop` and `Gfx_EndFrame`.

## One pass of `Sys_MainLoop` (0x80011800)

| site | what |
|---|---|
| top to `+0x348` | pause toggle, level requests (`Nsf_CloseLevel`/`Nsf_OpenLevel`), `Nsf_UpdatePaging(-1)` at 0x80011B48 |
| `+0x360` 0x80011B60 | `Cam_Update` when `g_Paused` (`4(gp)`) is 0 |
| `+0x368` 0x80011B68 | `Gfx_UpdateView`: a branch target, the first block of the picture half |
| after it | with `g_FrameFlags & 1` and zone items: `SetRotMatrix(0x80060774)`, the world copy into the scratchpad up to 0x1F800280, `func_8002FB84(0)`, `Gfx_DrawWorlds(0x8005F390, g_BuildBuffer[0], g_Scene, 0x80060944)`; else `Gfx_ResetOt(buffer + 0x18, 0x800)`; then `Gfx_DrawStreaks` / `func_80045218` |
| `+0x580` 0x80011D80 | `Obj_UpdateAll(g_Paused == 0)` (the argument is set in the delay slot) |
| `+0x58C` 0x80011D8C | `Sys_BeforeDrawObjects`, an empty function |
| `+0x594` 0x80011D94 | `Gfx_DrawObjects(*(g_BuildBuffer[0] + 0x2088), g_BuildBuffer[0], g_Scene)` |
| `+0x5B4` 0x80011DB4 | `Gfx_EndFrame` |
| `+0x5BC` 0x80011DBC | `func_80011E90`, the pass's transition |

The stretches 0x80011B68..0x80011D80 and 0x80011D94..0x80011DB4 hold their values in temporaries and `gp` only; no
saved register carries into or out of them.

## `Gfx_EndFrame` (0x8001645C)

Fade, `func_80016C58`, the letterbox bands (`Gfx_AddFillRect`), then `Snd_Update` at `+0x334` 0x80016790 (a block
start). After it the device tail: `DrawSync`, the buffer swap (`g_DrawBuffer` 0x80063454 and `g_BuildBuffer[0]`
0x80063458 trade), the VSync calls at 0x80016838 and 0x80016868, `DrawOTag(g_DrawBuffer + 0x18)` and
`Gfx_ResetFrame(build)`.

A GfxFrame holds its primitive base at +4 and cursor at +8, its ordering table at +0x18, the object queue's head at
+0x2088 and its node base and cursor at +0x208C/+0x2090. `Gfx_ResetFrame` (0x800163C4, through 0x800163B4) sets the
cursor to the base, the head to 0 and the node cursor to its base. The ordering table is cleared by the picture half
itself (`Gfx_DrawWorlds`, or `Gfx_ResetOt` without a world), so the half run again on a reset buffer rebuilds it.

## Objects

`g_ObjPool` (0x8006CB14) points at 0x60 records of 0x23C bytes (`Obj_InitPool` 0x8001EE64); free records are linked
from 0x8006CB58. The first word is 1 while the record is live (`Obj_Init`) and 0 after `Obj_Kill`. Offsets: +0x0C the
object's executable, +0x60 position (x, y, z), +0x6C rotation (three words, 4096 a turn), +0x78 scale (x negated to
flip), +0xE8 animation, +0xEC animation frame. The player's record (`g_Player`, 0x8005F38C) is not one of them: in
scene 0x6 and scene 0x15 it sits 4 bytes past the pool's last record.

`Obj_Update` (0x8001C718) runs an object's GOOL code and physics (`Obj_UpdatePhysics`) only while bit 0x8 of
`g_FrameFlags` is set: it is that bit's only reader in the executable. With the bit set, the argument `flag`, or an
object's state flag 0x100000 in its place, selects the logic path; otherwise the object only places itself
(`func_8001AA80`). Queueing is separate: an object is queued with `Gfx_QueueObject` (0x8001ABF8) when its kind's
draw bit is set, and a `flag` of 0 also needs `g_FrameFlags` 0x8000000 (set in levels) or the object's 0x100000.
`Obj_UpdateAll(0)` (the paused pass) therefore still runs the GOOL code of objects holding 0x100000. `Input_Update`
runs for the player either way.

`Obj_UpdateAll` walks with `Obj_ForEachList` (0x80018DD0) over the 8 list heads at 0x8006CB18, handing it
`func_80018CAC` and `Obj_Update`; `Obj_UpdateAll` is the only code that passes `Obj_Update` (0x8001C310 is its only
reference). `func_80018CAC` reads each child's next sibling (+0x48) before descending into it with `func_80018AA8`,
which runs the function on the node first, kills it (`Obj_Kill`) when it returns -0x19, then walks its children
(+0x4C) up to the first dead one. The lists change under the walk (spawns, `Obj_SetParent`, kills): in scene 0x15 an
object spawned during the walk is queued where the walk reaches it, not where it sits in the lists afterwards, some
objects are updated and queued twice in one walk, and an object queued and then killed is still drawn. The frame's
queue order is the walk's, not the lists' order once the walk is done. `Obj_Update`'s tail (0x8001CBE4) is the only call of
`Gfx_QueueObject`.

## Camera

`g_CameraX..Z` (0x800607F4, 24.8) and `g_CameraAngleX..Z` (0x80060800, 4096 a turn) are what `Gfx_UpdateView` reads.

`Gfx_UpdateView` steps two things when `g_Paused` is 0, so a second call in one pass draws them a tick further:

- `g_ScreenShake` (0x8006CB78, 24.8): `SetGeomOffset` takes 0x8006CC20 + 0x100 and 0x8006CC24 + 0x6C plus its whole part, then
  it steps toward 0 with alternating sign (s > 0 becomes 1 - s, s < 0 becomes -1 - s, 0 stays). It moves the
  picture, not the camera.
- The streaks' scroll in `Gfx_SetupScene` (scene flag 0x10): the u16 pair at 0x80062A74 and the word at 0x80062A78 are
  copied to 0x80062A6C/0x80062A70 and stepped by property 0x1B5's velocity (or 0x8006CCC4..CC while `g_FrameFlags`
  0x10000000 is set).

`Gfx_EndFrame` steps `g_FrameCount`, the counter at 0x80060944 and the fade at 0x8006CD18 before it draws them.

## Animation

`anim_frame` (+0xEC) holds the frame index << 8. Each drawn object's frame is fetched by `func_8001A90C` for
`func_8001A4A4`, which stores two vertex frames at +0x20 and +0x24 of the block `GoolObject.unk_8` points to (and
the index at +0x2C). Its callers in the C so far are `func_8001D63C` and the GOOL interpreter (`80037698.c`); with
`g_FrameFlags` 0x8 clear an object keeps the frames of its last update. An animation whose header word at +8 has bit 0 set stores every other frame: an even index
fetches stored frame index/2 twice, an odd one stored frames index/2 and index/2 + 1. `Gfx_QueueObject` (kind 1)
sends a pair whose two frames are equal to the static drawer (table 0x8005B708) and any other pair to the morph
drawer (table 0x8005B70C); a vertex frame names its model entry at +0x10. The morphing models' vertex
reader (`func_8003ED14`, table 0x8005B70C) decodes both frames' bit-packed vertex deltas in step ($s4 and $s5, the
control words at $gp) and adds the two decoded positions under one mask at 0x8003EF28: a fixed equal weight, with no
weight input. The retail drawer can draw only the halfway pose between two frames.

The morph vertex readers are subroutines with a register interface of their own: `func_8003ED14` (unlit; 16 `jal`
sites in the drawers at 0x8003EF88, `func_8003F1D8`, `func_8003F410` and `func_800417B0`) and a lit counterpart that opens the same way at
`func_80040D14` (3 sites). In: $t0 the strip command (bit 26 set: the vertex cached at $v1 + 0x140 + (($t0 >> 13) &
0x7F8)), $v1 the scratchpad, $s4 and $s5 the two frames' streams, $gp the field widths (0: three raw bytes a vertex),
$t3 and $sp the two bit buffers, $t4 the bit count, $t1 the x/y lane mask, $t2 the carry mask, $s6 the colours. Out:
$t9 the VXY pair, $t8 z (both also stored in the cache), $a0 the vertex's colour. The two frames' decoded bytes meet at
0x8003EF28: xy = ((a & $t1) + (b & $t1)) << 1 plus the box's xy (0x118), masked by $t2; z = ((a & 0xFF00) + (b &
0xFF00)) >> 7 plus the box's z (0x11C). A weight of k quarters on the second frame is the same sum as (4 - k)(a & $t1)
+ k(b & $t1) and ((4 - k)(a & 0xFF00) + k(b & 0xFF00)) >> 8, exact in each 16-bit lane, and k = 2 is the retail sum.
`Gfx_DrawObjects` calls a packet's place, prepare and draw steps (`jalr` $s6, $t8, $s7 at 0x8003DC94) with the packet
in $sp.

A kind 1 packet is 0x64 bytes: +0 the next packet, +4, +8 and +0xC the place, prepare and draw steps, +0x10 the object
(which `Gfx_DrawObjects` loads into $gp), +0x28 and +0x2C the block's two frames (+0x20 and +0x24). The morph draw
steps that reach a reader are 0x8003F1C8, 0x8003F1D8, 0x8003F410, 0x80040FC4 and 0x800417B0 (the table at 0x8005B708,
indexed by the object's byte +0x121, or the word at 0x80062ACC when that is 0); 0x8003EF88 has no caller of its own:
0x80040FC4 branches to 0x8003EF8C when the object's word +0x6C is 0, and otherwise reads through `func_80040D14`. Each
step runs `func_800436F4` before its first read, with the packet still in $sp: it loads $s4 and $s5 from +0x28 and +0x2C, sets $t1 to 0x00FF00FF and $t2 to 0xFFFEFFFF, and
stores the two frames' boxes halved at 0x118 and 0x11C (a frame's packed x/y at +0 and z at +4; the x/y words are added
as one word, its low half halved as a signed halfword and the word shifted right 17 for the high half). A vertex is
thus the mean of the two frames' own positions, four times the byte plus the frame's box, so a weight of k quarters
applies to the boxes as to the bytes: ((4 - k) box_a + k box_b) / 4.

`func_80040D14` differs from `func_8003ED14` only after the sum: it loads the vertex into VXY0/VZ0 through VXY2/VZ2,
runs three MVMVAs on the light matrix (0x4A4AE012 on V1 then IR2, 0x4A4A6012 on V0 then IR1, 0x4A4B6012 on V2 then
IR3) and caches (IR2 << 16 | IR1 & 0xFFFF, IR3). In the raw path ($gp 0) neither reader stores the running bytes at
0x110..0x116.
