# Camera cuts — Crash Bandicoot 2 (`SCUS_941.54`)

Where the camera jumps rather than moves, read from `Cam_Update` (0x80026CA0), `func_8002648C` and the mode handlers'
equivalent C in `src/game/80020A24.c`.

`Cam_Update` clears the byte at 0x8005B99B, processes the camera's messages (`Cam_ProcessMessages`), then runs the
handler of `g_CameraMode` (0x8005B994) when it is not 5 (following the path):

| mode | set by | handler | camera |
|---|---|---|---|
| 6 | `func_8002648C` message 3 | `func_80022A5C` | eases to the next path over 0x10 frames (0x8005BA14) from the position saved at the message |
| 7 | `func_8002648C` message 1 | `func_80022E74` | follows the current path until message 6 sets 0x8005B99B; then `Cam_MoveAlongPath` puts it on the prepared path of another zone (0x8005BAD8..0x8005BAE0) and the mode returns to 5: a cut |
| 9 | `func_8002648C` message 5 | `func_80022F58` | walks the zones' paths (`func_800267C4`) and puts it on the one found in the same call, then mode 5: a cut |

`Sys_LoadCheckpoint` (0x8002F6C8) restores the zone, the camera path and its progress from a continue block; the
camera is wherever the block put it: a cut. GOOL calls it (`Gool_OpMisc` case 12/1) as well as `Sys_MainLoop`.

Observed in psx-crash's run of the attract demos (scenes 0x1E and 0x21): `Sys_LoadCheckpoint` runs in the first frame of
each level; modes 7 and 9 were not reached there. Mode 7 is reached in scene 0x1C (the New Game intro) 118 frames after
the level opens, with the go byte set in the same frame: the picture cuts to black. A run through every scene and
every spawn point with no input, `g_CameraMode` watched, wrote modes 5 and 7 only; mode 9 was not reached.
