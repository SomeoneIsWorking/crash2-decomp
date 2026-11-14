# The level's spawn index — Crash Bandicoot 2 (`SCUS_941.54`)

`D_8006CD7C` is GOOL global 131 (`g_GoolGlobals` 0x8006CB70 + 131 × 4). `func_8001F9C4` starts a level at the spawn
point it names (`NsdLevel.spawns[slot]`), after picking the slot itself for one level.

## How `func_8001F9C4` picks the slot

- Level 2 (`NsdLevel.level_id`, the header word at 8), the warp room: from `D_8006CCD8` bit 0 (slot 4), else from the
  table at 0x8005B914 keyed by the level left (`D_8006CCD4`, else `D_8006CBC0`, `>> 8`, with the remaps in the
  switch): one more than the key's position, 3 when the key is not in it. The table has 31 keys; level 2 has 32
  spawn points.
- Any other level: the index as the game left it.
- Either way a slot at or past `NsdLevel.spawn_count` becomes 0.

## Every store to the index

| writer | value |
|---|---|
| `func_8001F9C4` (0x8001FBF0) | the slot above |
| `Sys_ResetGameInfo` (0x8002F108) | 0 |
| `Sys_MainLoop` (0x80011A54), a demo level's end | 0 |
| GOOL `GVAW` (opcode 0x20) with global 131 | 0 or 1 |

The executable holds no other load or store of the word (the instructions whose offset is 0xCD7C). The GOOL stores
were found by reading every GOOL entry (type 11) of every NSF on the disc for opcode 0x20 whose index operand is the
literal 131 or a constant-pool word of 131 << 8:

| GOOL entry | levels | value |
|---|---|---|
| `WillC` (0x7492AACD, the player) | all but 0x28, 0x29, 0x3B, 0x3C | 1 (pool word) |
| `CHeOC` (0x4D59D94D) | 2 | 0 at its code's second word; 1 further on |
| `DispC` (0x4E938CCD) | all | 0, with global 17 (`D_8006CBB4`) set to 0x600, the demo mode |
| `CardC` (0x4C5366CD) | 2 | 0 |
| `CortC` (0x4CC36ECD) | 7 | 0 |
| `WinOC` (0x7492F94D) | 0x28, 0x29 | 0 |
| `GamOC` (0x5452D94D) | 0x3B | 0 |

A store watch on the word, entering level 2 with the level left set to key 31, shows `func_8001F9C4` storing 6 and,
on the same frame, a `CHeOC` object's GOOL storing 0. So the warp room's slot does not outlive its start, and every
other level starts at spawn 0 or 1. Level 2 never starts at the index the game left.

## A spawn point no start reaches

Level 0x1D has 8 spawn points. `WillC`'s code asks `Gool_OpChunk` kind 3 (`Nsf_CloseEntry(src, 0)`, is the entry
resident) for `Cda0V` (0x4C694073, its pool word 41) beside `Cda0V`'s neighbour 0x4C694ED5. Level 0x1D's NSD has no
record for 0x4C694073 (neither have levels 2, 7, 9, 0x12, 0x1A, 0x22, 0x25, 0x28, 0x29, 0x3B, 0x3C), and
`Nsf_LookupEid` walks a bucket with no bound, so the query runs off the records. Started at spawn 6 with up held the
player reaches that query in 151 frames; started at spawn 0 or 1 it does not in 300.
