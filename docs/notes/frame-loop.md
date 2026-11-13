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
flip), +0xE8 animation, +0xEC animation frame.

`Obj_Update` (0x8001C718) runs an object's GOOL code and physics (`Obj_UpdatePhysics`) only while bit 0x8 of
`g_FrameFlags` is set: it is that bit's only reader in the executable. With the bit set, the argument `flag`, or an
object's state flag 0x100000 in its place, selects the logic path; otherwise the object only places itself
(`func_8001AA80`). Queueing is separate: an object is queued with `Gfx_QueueObject` (0x8001ABF8) when its kind's
draw bit is set, and a `flag` of 0 also needs `g_FrameFlags` 0x8000000 (set in levels) or the object's 0x100000.
`Obj_UpdateAll(0)` (the paused pass) therefore still runs the GOOL code of objects holding 0x100000. `Input_Update`
runs for the player either way.

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
the index at +0x2C). An animation whose header word at +8 has bit 0 set stores every other frame: an even index
fetches stored frame index/2 twice, an odd one stored frames index/2 and index/2 + 1. Which drawer reads the pair,
and at what weight it blends them, is not yet read.
