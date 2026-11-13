# Our names (`SCUS_941.54`)

Names the psx-crash port uses for this executable where they differ from or go beyond this tree's own. The symbol map
(`config/symbol_addrs.txt`) keeps the tree's names; ours enter it only where the tree has none. "Port source" is the
psx-crash file and key that holds the address (`titles/<title>/executable.json` paths, or `entry/facts.cpp` initializers);
the port adopts the map's name when its constants are generated (psx-crash `docs/plans/0001-addresses-from-the-decomp-symbol-map.md`).

## Names that differ

| address | this tree | psx-crash | port source |
|---|---|---|---|
| `0x800117BC` | `main` | `NativeBootCMain` | `runtime.native_boot.c_main` |
| `0x80011800` | `Sys_MainLoop` | `NativeFrameCoreLoop` | `runtime.native_frame.core_loop` |
| `0x80011E90` | `func_80011E90` | `NativeFrameTransition` | `runtime.native_frame.transition` |
| `0x80014C68` | `Nsf_OpenLevel` | `SceneLoadEntry` | `runtime.native_frame.scene.load_entry` |
| `0x80015614` | `Sys_Init` | `NativeBootInit` | `runtime.native_boot.init` |
| `0x8001645C` | `Gfx_EndFrame` | `NativeFrameGpuUpdate` | `runtime.native_frame.gpu_update` |
| `0x80036164` | `Save_InitCard` | `NativeBootCallbackInitialize` | `runtime.native_boot.callback_initialize` |
| `0x8003BE88` | `Sys_TimerHandler` | `RootCounterIncrement` | `runtime.native_frame.root_counter_increment` |
| `0x80046B50` | `CdControl` | `NativeBootCdControl` | `runtime.native_boot.cd_control` |
| `0x80046C88` | `CdControlF` | `NativeBootCdControlF` | `runtime.native_boot.cd_control_f` |
| `0x800476A8` | `CD_sync` | `NativeBootCdSync` | `runtime.native_boot.cd_sync` |
| `0x800482BC` | `CD_init` | `NativeBootCdInitialize` | `runtime.native_boot.cd_initialize` |
| `0x80048F74` | `CdRead` | `NativeBootCdRead` | `runtime.native_boot.cd_read` |
| `0x80049078` | `CdReadSync` | `NativeBootCdReadSync` | `runtime.native_boot.cd_read_sync` |
| `0x80049244` | `printf` | `GpuWatchdogLog` | `entry/facts.cpp kGpuWatchdog.log` |
| `0x80049B2C` | `__SN_ENTRY_POINT` | `Crt0Entry` | `entry/facts.cpp kCrt0.entry` |
| `0x80049BD4` | `__main` | `GameMainEntry` | `runtime.game_main_entry` |
| `0x80049CAC` | `_bu_init` | `CallbackBootChangeClearPad` | `entry/facts.cpp kCallbackBoot.changeClearPad` |
| `0x80049CCC` | `OpenEvent` | `CallbackBootOpenEvent` | `entry/facts.cpp kCallbackBoot.openEvent` |
| `0x80049CFC` | `EnableEvent` | `CallbackBootCloseEvent` | `entry/facts.cpp kCallbackBoot.closeEvent` |
| `0x80049D1C` | `EnterCriticalSection` | `FirstSyscallEntry` | `runtime.first_syscall_entry` |
| `0x80049D2C` | `ExitCriticalSection` | `CallbackBootExitCritical` | `entry/facts.cpp kCallbackBoot.exitCritical` |
| `0x80049DCC` | `SetRCnt` | `SetRootCounter` | `runtime.native_frame.set_root_counter` |
| `0x80049EA0` | `StartRCnt` | `StartRootCounter` | `runtime.native_frame.start_root_counter` |
| `0x80049ED4` | `StopRCnt` | `StopRootCounter` | `runtime.native_frame.stop_root_counter` |
| `0x8004A484` | `VSync` | `Vsync` | `runtime.vsync` |
| `0x8004A7F4` | `SetIntrMask` | `GpuWatchdogCritical` | `entry/facts.cpp kGpuWatchdog.critical` |
| `0x8004E970` | `set_alarm` | `NativeBootGpuWatchdogStart` | `runtime.native_boot.gpu_watchdog_start` |
| `0x8004E9A4` | `get_alarm` | `NativeBootGpuWatchdogCheck` | `runtime.native_boot.gpu_watchdog_check` |
| `0x8005B030` | `InitCARD` | `CallbackBootInitializePad` | `entry/facts.cpp kCallbackBoot.initializePad` |
| `0x8005B088` | `StartCARD` | `CallbackBootStartPad` | `entry/facts.cpp kCallbackBoot.startPad` |
| `0x8005F1A4` | `g_UseCd` | `NativeBootUseCdAddress` | `runtime.native_boot.use_cd_address` |
| `0x8005F1D0` | `g_Quit` | `NativeFrameDone` | `runtime.native_frame.done.address` |
| `0x8005F3EC` | `D_8005F3EC` | `SceneIdAddress` | `runtime.native_frame.scene.id_address` |
| `0x8005F3F0` | `g_LevelRequest` | `SceneRequestAddress` | `runtime.native_frame.scene.request_address` |
| `0x80069034` | `g_LevelFiles` | `FrameSceneTable` | `entry/facts.cpp kFrame.sceneTable` |
| `0x80069920` | `g_Pads` | `PadBuffersSlot0Buffer` | `entry/facts.cpp kPadBuffers.slot0Buffer` |

`func_`/`D_` names in the first column are placeholders this tree's C already refers to; naming them in the map would
break its build, so ours waits here until the tree renames them.

## Names added to the map

| address | added as | kind | port source |
|---|---|---|---|
| `0x80011040` | `GpuWatchdogDiagnosticFormat` | data | `entry/facts.cpp kGpuWatchdog.diagnosticFormat` |
| `0x80011074` | `GpuWatchdogCallbackFormat` | data | `entry/facts.cpp kGpuWatchdog.callbackFormat` |
| `0x8005C7CC` | `LibcdInitLastSyncCallback` | data | `entry/facts.cpp kLibcdInit.lastSyncCallback` |
| `0x8005C7D0` | `LibcdInitLastReadyCallback` | data | `entry/facts.cpp kLibcdInit.lastReadyCallback` |
| `0x8005C7DC` | `LibcdInitLastStatus` | data | `entry/facts.cpp kLibcdInit.lastStatus` |
| `0x8005C7E0` | `LibcdInitLastResult` | data | `entry/facts.cpp kLibcdInit.lastResult` |
| `0x8005C7EC` | `LibcdInitLastCommand` | data | `entry/facts.cpp kLibcdInit.lastCommand` |
| `0x8005C7ED` | `LibcdInitPendingCommand` | data | `entry/facts.cpp kLibcdInit.pendingCommand` |
| `0x8005CAAC` | `LibcdInitSyncStatus` | data | `entry/facts.cpp kLibcdInit.syncStatus` |
| `0x8005DC98` | `VsyncQueryCounter` | data | `runtime.vsync.query_counter` |
| `0x8005DDF0` | `GpuWatchdogGp0Pointer` | data | `entry/facts.cpp kGpuWatchdog.gp0Pointer` |
| `0x8005DDF4` | `GpuWatchdogStatusPointer` | data | `entry/facts.cpp kGpuWatchdog.statusPointer` |
| `0x8005DDFC` | `GpuWatchdogGp1Pointer` | data | `entry/facts.cpp kGpuWatchdog.gp1Pointer` |
| `0x8005DE0C` | `GpuWatchdogDmaControlPointer` | data | `entry/facts.cpp kGpuWatchdog.dmaControlPointer` |
| `0x8005DE10` | `GpuWatchdogLastCallback` | data | `entry/facts.cpp kGpuWatchdog.lastCallback` |
| `0x8005DE14` | `GpuWatchdogLastPayload` | data | `entry/facts.cpp kGpuWatchdog.lastPayload` |
| `0x8005DE18` | `GpuWatchdogLastArgument` | data | `entry/facts.cpp kGpuWatchdog.lastArgument` |
| `0x8005DE20` | `GpuWatchdogQueueWrite` | data | `entry/facts.cpp kGpuWatchdog.queueWrite` |
| `0x8005DE24` | `GpuWatchdogQueueRead` | data | `entry/facts.cpp kGpuWatchdog.queueRead` |
| `0x8005DE30` | `GpuWatchdogCriticalToken` | data | `entry/facts.cpp kGpuWatchdog.criticalToken` |
| `0x8005DE34` | `GpuWatchdogDeadline` | data | `entry/facts.cpp kGpuWatchdog.deadline` |
| `0x8005DE38` | `GpuWatchdogPollCount` | data | `entry/facts.cpp kGpuWatchdog.pollCount` |
| `0x8006996C` | `PadBuffersSlot1Buffer` | data | `entry/facts.cpp kPadBuffers.slot1Buffer` |
| `0x80011E88` | `Sys_BeforeDrawObjects` | func | `frame/draw_replay.cpp kSampleStub` |

## Code sites

Addresses inside a function the port stops at, returns to or patches; not symbols.

| address | where | port source |
|---|---|---|
| `0x80011848` | `NativeFrameCoreLoop + 0x48` | `runtime.native_frame.iteration.entry` |
| `0x80011B68` | `NativeFrameCoreLoop + 0x368` | `frame/draw_section.cpp kViewCall` |
| `0x80011D80` | `NativeFrameCoreLoop + 0x580` | `frame/draw_section.cpp kObjectsCall` |
| `0x80011D94` | `NativeFrameCoreLoop + 0x594` | `frame/draw_section.cpp kDrawObjects` |
| `0x80011DB4` | `NativeFrameCoreLoop + 0x5B4` | `frame/draw_section.cpp kEndFrameCall` |
| `0x80016790` | `NativeFrameGpuUpdate + 0x334` | `frame/draw_section.cpp kSoundUpdate` |
| `0x80011DC4` | `NativeFrameCoreLoop + 0x5C4` | `runtime.native_frame.done.load_entry` |
| `0x80016838` | `NativeFrameGpuUpdate + 0x3DC` | `runtime.native_frame.first_vsync` |
| `0x80016840` | `NativeFrameGpuUpdate + 0x3E4` | `runtime.native_frame.after_first_vsync` |
| `0x80016868` | `NativeFrameGpuUpdate + 0x40C` | `runtime.native_frame.second_vsync` |
| `0x80016870` | `NativeFrameGpuUpdate + 0x414` | `runtime.native_frame.after_vsync` |
