# Static Analysis Report

Date: 2026-11-04. Branch analyzed: `exercise-1` (FreeRTOS-Kernel).

## Scope and setup

- **Files:** `tasks.c`, `queue.c`, `list.c`, `timers.c`, `event_groups.c`, `stream_buffer.c`, `portable/MemMang/heap_4.c`, `portable/MSVC-MingW/port.c`
- **Config:** `examples/sample_configuration/FreeRTOSConfig.h` with `configTICK_TYPE_WIDTH_IN_BITS` changed to 32-bit (the MSVC-MingW port rejects 64-bit). Includes: `include/`, `portable/MSVC-MingW`.
- **Target:** x86-64 Windows, MSVC 14.51 headers.
- **Tools:** PVS-Studio 8.01, Cppcheck (`--enable=warning,style,performance,portability`), clang-tidy (`clang-analyzer-*,bugprone-*`), Cppcheck Premium 26.9.0 (`--premium=bughunting`).

## PVS-Studio

| Level   | Rule  | File    | Line | Message                                                                                           |
| ------- | ----- | ------- | ---- | ------------------------------------------------------------------------------------------------- |
| warning | V557  | tasks.c | 3480 | Array overrun is possible. The value of 'xIdleTaskNameIndex' index could reach 15.                |
| note    | V1008 | tasks.c | 3150 | Consider inspecting the 'for' operator. No more than one iteration of the loop will be performed. |
| note    | V1008 | tasks.c | 3496 | Consider inspecting the 'for' operator. No more than one iteration of the loop will be performed. |

## Cppcheck

| Severity | Check                     | File            | Line | Message                                                                                                                                             |
| -------- | ------------------------- | --------------- | ---- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| error    | arrayIndexOutOfBounds     | tasks.c         | 3604 | Array '"IDLE"[5]' accessed at index 14, which is out of bounds.                                                                                     |
| warning  | arrayIndexOutOfBoundsCond | tasks.c         | 5273 | Either the condition '(uxTopPriority)==0' is redundant or the array 'pxReadyTasksLists[5]' is accessed at index 4294967295, which is out of bounds. |
| warning  | ctunullpointer            | list.c          | 150  | Null pointer dereference: pxList                                                                                                                    |
| warning  | ctunullpointer            | stream_buffer.c | 1018 | Null pointer dereference: pxStreamBuffer                                                                                                            |
| warning  | ctunullpointer            | stream_buffer.c | 1583 | Null pointer dereference: pxStreamBuffer                                                                                                            |
| style    | knownConditionTrueFalse   | tasks.c         | 4858 | Condition 'xConstTickCount==(TickType_t)0U' is always false                                                                                         |
| style    | unreadVariable            | event_groups.c  | 556  | Variable 'xMatchFound' is assigned a value that is never used.                                                                                      |

## Cppcheck Premium

Cppcheck Premium 26.9.0, `--premium=bughunting --enable=warning,style,performance,portability --platform=win64`, same files and includes (config: `examples/template_configuration/FreeRTOSConfig.h`, 32-bit ticks). 155 raw findings; the table keeps only those that can actually fail.

| Severity | Check                                     | File                       | Line | Message                                                                                     | Why it is possible                                                                                                                                                                                                     |
| -------- | ----------------------------------------- | -------------------------- | ---- | ------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| warning  | premium-bughuntingPointerSubtractionOverflow | portable/MSVC-MingW/port.c | 241  | Cannot determine that pointer subtraction can not overflow: pcTopOfStack-sizeof(struct ThreadState_t) | `ThreadState_t` (16 bytes) is written below the top of the stack with no size check. A task created with `uxStackDepth` of 1-2 words writes before the buffer; the `configASSERT` at `tasks.c:2002` only runs after the write. |
| warning  | premium-bughuntingArrayIndexOutOfBounds   | tasks.c                    | 1878 | Cannot determine that array index is valid: pcName[x]                                       | The copy reads `pcName` until NUL or `configMAX_TASK_NAME_LEN`. A name buffer that is shorter than that and not NUL-terminated is over-read.                                                                            |
| warning  | premium-bughuntingBufferOverflow          | tasks.c                    | 1835 | When calling 'memset' it cannot be determined that 1st argument is not overflowed           | With `xTaskCreateStatic`, the stack is filled for `uxStackDepth * sizeof(StackType_t)` bytes, but nothing checks that `puxStackBuffer` really has that size.                                                            |
| error    | premium-bughuntingUninit                  | tasks.c                    | 3668 | Cannot determine that 'uxIdleTaskStackSize' is initialized                                  | `uxIdleTaskStackSize` is uninitialized and only set by the application's `vApplicationGetIdleTaskMemory()`. A hook that skips it passes garbage to `xTaskCreateStatic`.                                                   |
| error    | premium-bughuntingUninit                  | timers.c                   | 294  | Cannot determine that 'uxTimerTaskStackSize' is initialized                                 | Same pattern with `vApplicationGetTimerTaskMemory()`.                                                                                                                                                                  |

Excluded as guaranteed false positives:

- **nullPointerRedundantCheck (55):** every one follows a `configASSERT(p != NULL)` (or `configASSERT(p)`). In this config, `configASSERT` disables interrupts and loops forever, so the dereference can't be reached with NULL. These replace the free Cppcheck `ctunullpointer` hits at `list.c:150` and `stream_buffer.c:1018`/`1583`.
- **arrayIndexOutOfBounds `tasks.c:3604` / arrayIndexOutOfBoundsCond `tasks.c:5273`:** same as free Cppcheck. The idle-name loop breaks on the NUL at index 4, and `taskSELECT_HIGHEST_PRIORITY_TASK` asserts `uxTopPriority` before decrementing.
- **bughuntingUninit / UninitNonConstArg (other 31):** output parameters filled by the callee (`prvGetNextExpireTime`, `prvSampleTimeNow`, `prvReadBytesFromBuffer`), buffers zeroed by `memset` right after allocation, `pvObjectList` set before `WaitForMultipleObjects`, `(void) ulErrorCode` casts, and `pxCurrentTCB`'s first word (always `pxTopOfStack`).
- **bughunting pointer/buffer overflow in `heap_4.c`, `queue.c`, `stream_buffer.c`, `tasks.c:1298`/`5259`:** each one is guarded, by `heapADD_WILL_OVERFLOW`/`heapSUBTRACT_WILL_UNDERFLOW`, the `SIZE_MAX / uxLength` check in `xQueueGenericReset`, the `configASSERT` bounds before each `memcpy`, the `sizeof(StaticTask_t) == sizeof(TCB_t)` assert, or the minimum stack size.
- **style (37):** `variableScope`, `constVariablePointer`, `unusedStructMember` and similar. These aren't errors.

## clang-tidy

| Severity | Check                               | File                       | Line | Message                                                                                                                  |
| -------- | ----------------------------------- | -------------------------- | ---- | ------------------------------------------------------------------------------------------------------------------------ |
| warning  | clang-analyzer-core.NullDereference | tasks.c                    | 4103 | Access to field 'pxIndex' results in a dereference of a null pointer (loaded from variable 'pxList')                     |
| warning  | bugprone-narrowing-conversions      | portable/MSVC-MingW/port.c | 193  | narrowing conversion from 'TickType_t' (aka 'unsigned int') to signed type 'LONG' (aka 'long') is implementation-defined |

## Cross-tool agreement

`tasks.c` around lines 3480-3604 (idle task name copy) is flagged by both PVS-Studio (V557) and Cppcheck (`arrayIndexOutOfBounds`). That makes it the strongest lead. The name is copied in a bounded loop, so it is probably safe, but both tools may be wrong about the loop bound.
