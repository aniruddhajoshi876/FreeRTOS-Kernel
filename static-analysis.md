# Static Analysis Report

Date: 2026-11-04. Branch analyzed: `exercise-1` (FreeRTOS-Kernel).

## Scope and setup

- **Files:** `tasks.c`, `queue.c`, `list.c`, `timers.c`, `event_groups.c`, `stream_buffer.c`, `portable/MemMang/heap_4.c`, `portable/MSVC-MingW/port.c`
- **Config:** `examples/sample_configuration/FreeRTOSConfig.h` with `configTICK_TYPE_WIDTH_IN_BITS` changed to 32-bit (the MSVC-MingW port rejects 64-bit). Includes: `include/`, `portable/MSVC-MingW`.
- **Target:** x86-64 Windows, MSVC 14.51 headers.
- **Tools:** PVS-Studio 8.01, Cppcheck (`--enable=warning,style,performance,portability`), clang-tidy (`clang-analyzer-*,bugprone-*`).

## Filtering

The raw run produced 207 findings (PVS-Studio 80, Cppcheck 93, clang-tidy 34). Most were noise and have been removed:

- `configASSERT` / `pdFALSE` macro patterns (PVS V562, Cppcheck `nullPointerRedundantCheck`)
- Defensive `!= NULL` checks that are dead only in this configuration (PVS V547/V560)
- Style opinions (`variableScope`, `constVariablePointer`, `unusedStructMember`, `bugprone-signed-bitwise`, `bugprone-macro-parentheses`, `bugprone-branch-clone`, `bugprone-reserved-identifier`)
- `memcpy`/`memset` warnings (`insecureAPI`), "potentially infinite loop" (V776), and Windows-simulator API advice (V513, V720)
- Intentional kernel idioms (V641 on `xListEnd`, V1059 macro `static` override)

The 12 findings below are the ones that could plausibly be real. None is confirmed. Each needs a read of the code and likely a check against the real project configuration.

## PVS-Studio

| Level | Rule | File | Line | Message |
|---|---|---|---|---|
| warning | V557 | tasks.c | 3480 | Array overrun is possible. The value of 'xIdleTaskNameIndex' index could reach 15. |
| note | V1008 | tasks.c | 3150 | Consider inspecting the 'for' operator. No more than one iteration of the loop will be performed. |
| note | V1008 | tasks.c | 3496 | Consider inspecting the 'for' operator. No more than one iteration of the loop will be performed. |

## Cppcheck

| Severity | Check | File | Line | Message |
|---|---|---|---|---|
| error | arrayIndexOutOfBounds | tasks.c | 3604 | Array '"IDLE"[5]' accessed at index 14, which is out of bounds. |
| warning | arrayIndexOutOfBoundsCond | tasks.c | 5273 | Either the condition '(uxTopPriority)==0' is redundant or the array 'pxReadyTasksLists[5]' is accessed at index 4294967295, which is out of bounds. |
| warning | ctunullpointer | list.c | 150 | Null pointer dereference: pxList |
| warning | ctunullpointer | stream_buffer.c | 1018 | Null pointer dereference: pxStreamBuffer |
| warning | ctunullpointer | stream_buffer.c | 1583 | Null pointer dereference: pxStreamBuffer |
| style | knownConditionTrueFalse | tasks.c | 4858 | Condition 'xConstTickCount==(TickType_t)0U' is always false |
| style | unreadVariable | event_groups.c | 556 | Variable 'xMatchFound' is assigned a value that is never used. |

## clang-tidy

| Severity | Check | File | Line | Message |
|---|---|---|---|---|
| warning | clang-analyzer-core.NullDereference | tasks.c | 4103 | Access to field 'pxIndex' results in a dereference of a null pointer (loaded from variable 'pxList') |
| warning | bugprone-narrowing-conversions | portable/MSVC-MingW/port.c | 193 | narrowing conversion from 'TickType_t' (aka 'unsigned int') to signed type 'LONG' (aka 'long') is implementation-defined |

## Cross-tool agreement

`tasks.c` around lines 3480-3604 (idle task name copy) is flagged by both PVS-Studio (V557) and Cppcheck (`arrayIndexOutOfBounds`). That makes it the strongest lead. The name is copied in a bounded loop, so it is probably safe, but both tools may be wrong about the loop bound.
