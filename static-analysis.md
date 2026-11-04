# Static Analysis Report

Date: 2026-11-04. Branch analyzed: `exercise-1` (FreeRTOS-Kernel).

## Scope and setup

- **Files:** `tasks.c`, `queue.c`, `list.c`, `timers.c`, `event_groups.c`, `stream_buffer.c`, `portable/MemMang/heap_4.c`, `portable/MSVC-MingW/port.c`
- **Config:** `examples/sample_configuration/FreeRTOSConfig.h` with `configTICK_TYPE_WIDTH_IN_BITS` changed to 32-bit (the MSVC-MingW port rejects 64-bit). Includes: `include/`, `portable/MSVC-MingW`.
- **Target:** x86-64 Windows, MSVC 14.51 headers.
- **Tools:** PVS-Studio 8.01 (CompilerCommandsAnalyzer), Cppcheck 2.x (`--enable=warning,style,performance,portability`), clang-tidy (`clang-analyzer-*,bugprone-*`, minus `bugprone-easily-swappable-parameters`).
- Most findings come from FreeRTOS macro patterns (`pdFALSE`/`pdTRUE`, `configASSERT`) and defensive checks, so many are likely false positives for a single configuration. Nothing here has been triaged as a confirmed bug.

## Summary

| Tool | Findings |
|---|---|
| PVS-Studio | 80 (21 error, 11 warning, 48 note) |
| Cppcheck | 93 |
| clang-tidy | 34 |

## PVS-Studio

| Rule | Count |
|---|---|
| V776 | 23 |
| V562 | 21 |
| V1027 | 11 |
| V547 | 7 |
| V641 | 6 |
| V560 | 4 |
| V513 | 3 |
| V1008 | 2 |
| V1059 | 1 |
| V720 | 1 |
| V557 | 1 |

| Level | Rule | File | Line | Message |
|---|---|---|---|---|
| error | V562 | event_groups.c | 199 | It's odd to compare 0 or 1 with a value of 0. |
| error | V562 | event_groups.c | 200 | It's odd to compare 0 or 1 with a value of 0: (uxBitsToWaitFor != 0) == 0. |
| error | V562 | event_groups.c | 330 | It's odd to compare 0 or 1 with a value of 0. |
| error | V562 | event_groups.c | 331 | It's odd to compare 0 or 1 with a value of 0: (uxBitsToWaitFor != 0) == 0. |
| error | V562 | event_groups.c | 485 | It's odd to compare 0 or 1 with a value of 0. |
| error | V562 | event_groups.c | 562 | It's odd to compare 0 or 1 with a value of 0: ((uxBitsToSet & 0xff000000UL) == 0) == 0. |
| error | V562 | heap_4.c | 237 | It's odd to compare 0 or 1 with a value of 0. |
| error | V562 | heap_4.c | 269 | It's odd to compare 0 or 1 with a value of 0. |
| error | V562 | heap_4.c | 278 | It's odd to compare 0 or 1 with a value of 0. |
| error | V562 | heap_4.c | 343 | It's odd to compare 0 or 1 with a value of 0. |
| error | V562 | heap_4.c | 363 | It's odd to compare 0 or 1 with a value of 0. |
| error | V562 | heap_4.c | 366 | It's odd to compare 0 or 1 with a value of 0. |
| error | V562 | queue.c | 1345 | It's odd to compare 0 or 1 with a value of 0: (pxQueue->uxItemSize == 0) == 0. |
| error | V562 | queue.c | 1667 | It's odd to compare 0 or 1 with a value of 0: (pxQueue->uxItemSize == 0) == 0. |
| error | V562 | queue.c | 2139 | It's odd to compare 0 or 1 with a value of 0: (pxQueue->uxItemSize != 0) == 0. |
| error | V562 | stream_buffer.c | 345 | It's odd to compare 0 or 1 with a value of 0: (xBufferSizeBytes > 0) == 0. |
| error | V562 | stream_buffer.c | 1078 | It's odd to compare 0 or 1 with a value of 0: (xBytesAvailable == 0) == 0. |
| error | V562 | tasks.c | 2284 | It's odd to compare 0 or 1 with a value of 0: (uxSchedulerSuspended == 0) == 0. |
| error | V562 | tasks.c | 3187 | It's odd to compare 0 or 1 with a value of 0: (uxSchedulerSuspended == 0) == 0. |
| error | V562 | timers.c | 425 | It's odd to compare 0 or 1 with a value of 0: ((xTimerPeriodInTicks > 0)) == 0. |
| error | V562 | timers.c | 1034 | It's odd to compare 0 or 1 with a value of 0. |
| warning | V547 | event_groups.c | 103 | Expression 'pxEventBits != NULL' is always true. |
| warning | V720 | port.c | 469 | It is advised to utilize the 'SuspendThread' function only when developing a debugger (see documentation for details). |
| warning | V560 | queue.c | 310 | A part of conditional expression is always true: (pxQueue != NULL). |
| warning | V560 | queue.c | 388 | A part of conditional expression is always true: (pxStaticQueue != NULL). |
| warning | V547 | queue.c | 872 | Expression 'xHandle != NULL' is always true. |
| warning | V560 | stream_buffer.c | 453 | A part of conditional expression is always true: (pxStaticStreamBuffer != NULL). |
| warning | V560 | stream_buffer.c | 453 | A part of conditional expression is always true: (pucStreamBufferStorageArea != NULL). |
| warning | V557 | tasks.c | 3480 | Array overrun is possible. The value of 'xIdleTaskNameIndex' index could reach 15. |
| warning | V547 | timers.c | 399 | Expression 'pxNewTimer != NULL' is always true. |
| warning | V547 | timers.c | 474 | Expression 'xCommandID < ((BaseType_t) 6)' is always true. |
| warning | V547 | timers.c | 525 | Expression 'xCommandID >= ((BaseType_t) 6)' is always true. |
| note | V1027 | event_groups.c | 101 | Pointer to an object of the 'xSTATIC_EVENT_GROUP' class is cast to unrelated 'EventGroupDef_t' class. |
| note | V641 | event_groups.c | 565 | The size of the '& ((pxList)->xListEnd)' buffer is not a multiple of the element size of the type 'ListItem_t const'. |
| note | V641 | event_groups.c | 665 | The size of the '& (pxTasksWaitingForBits->xListEnd)' buffer is not a multiple of the element size of the type 'const ListItem_t'. |
| note | V1027 | event_groups.c | 713 | Pointer to an object of the 'EventGroupDef_t' class is cast to unrelated 'xSTATIC_EVENT_GROUP' class. |
| note | V776 | heap_4.c | 245 | Potentially infinite loop. |
| note | V776 | heap_4.c | 261 | Potentially infinite loop. |
| note | V776 | heap_4.c | 269 | Potentially infinite loop. |
| note | V776 | heap_4.c | 278 | Potentially infinite loop. |
| note | V776 | heap_4.c | 343 | Potentially infinite loop. |
| note | V776 | heap_4.c | 362 | Potentially infinite loop. |
| note | V776 | heap_4.c | 363 | Potentially infinite loop. |
| note | V776 | heap_4.c | 364 | Potentially infinite loop. |
| note | V776 | heap_4.c | 506 | Potentially infinite loop. |
| note | V641 | list.c | 57 | The size of the '& (pxList->xListEnd)' buffer is not a multiple of the element size of the type 'ListItem_t'. |
| note | V641 | list.c | 67 | The size of the '& (pxList->xListEnd)' buffer is not a multiple of the element size of the type 'ListItem_t'. |
| note | V641 | list.c | 68 | The size of the '& (pxList->xListEnd)' buffer is not a multiple of the element size of the type 'ListItem_t'. |
| note | V641 | list.c | 193 | The size of the '& (pxList->xListEnd)' buffer is not a multiple of the element size of the type 'ListItem_t'. |
| note | V513 | port.c | 250 | Use _beginthreadex/_endthreadex functions instead of CreateThread/ExitThread functions. |
| note | V513 | port.c | 330 | Use _beginthreadex/_endthreadex functions instead of CreateThread/ExitThread functions. |
| note | V513 | port.c | 578 | Use _beginthreadex/_endthreadex functions instead of CreateThread/ExitThread functions. |
| note | V776 | queue.c | 361 | Potentially infinite loop. |
| note | V1027 | queue.c | 411 | Pointer to an object of the 'xSTATIC_QUEUE' class is cast to unrelated 'QueueDefinition' class. |
| note | V1027 | queue.c | 462 | Pointer to an object of the 'QueueDefinition' class is cast to unrelated 'xSTATIC_QUEUE' class. |
| note | V1027 | stream_buffer.c | 409 | Pointer to an object of the 'xSTATIC_STREAM_BUFFER' class is cast to unrelated 'StreamBufferDef_t' class. |
| note | V1027 | stream_buffer.c | 469 | Pointer to an object of the 'xSTATIC_STREAM_BUFFER' class is cast to unrelated 'StreamBufferDef_t' class. |
| note | V1027 | stream_buffer.c | 501 | Pointer to an object of the 'StreamBufferDef_t' class is cast to unrelated 'xSTATIC_STREAM_BUFFER' class. |
| note | V547 | stream_buffer.c | 908 | Expression is always true. |
| note | V776 | stream_buffer.c | 1078 | Potentially infinite loop. |
| note | V776 | stream_buffer.c | 1365 | Potentially infinite loop. |
| note | V547 | stream_buffer.c | 1404 | Expression 'xFirstLength <= xCount' is always true. |
| note | V1059 | tasks.c | 144 | Macro name overrides the 'static' keyword. This may lead to undefined behavior. |
| note | V1027 | tasks.c | 1274 | Pointer to an object of the 'xSTATIC_TCB' class is cast to unrelated 'tskTaskControlBlock' class. |
| note | V776 | tasks.c | 1811 | Potentially infinite loop. |
| note | V776 | tasks.c | 1865 | Potentially infinite loop. |
| note | V776 | tasks.c | 2284 | Potentially infinite loop. |
| note | V1008 | tasks.c | 3150 | Consider inspecting the 'for' operator. No more than one iteration of the loop will be performed. |
| note | V776 | tasks.c | 3187 | Potentially infinite loop. |
| note | V1008 | tasks.c | 3496 | Consider inspecting the 'for' operator. No more than one iteration of the loop will be performed. |
| note | V1027 | tasks.c | 4322 | Pointer to an object of the 'tskTaskControlBlock' class is cast to unrelated 'xSTATIC_TCB' class. |
| note | V776 | tasks.c | 5313 | Potentially infinite loop. |
| note | V776 | tasks.c | 5399 | Potentially infinite loop. |
| note | V776 | tasks.c | 7821 | Potentially infinite loop. |
| note | V776 | tasks.c | 7957 | Potentially infinite loop. |
| note | V776 | tasks.c | 8072 | Potentially infinite loop. |
| note | V776 | timers.c | 332 | Potentially infinite loop. |
| note | V1027 | timers.c | 397 | Pointer to an object of the 'xSTATIC_TIMER' class is cast to unrelated 'tmrTimerControl' class. |
| note | V1027 | timers.c | 667 | Pointer to an object of the 'tmrTimerControl' class is cast to unrelated 'xSTATIC_TIMER' class. |
| note | V776 | timers.c | 760 | Potentially infinite loop. |

## Cppcheck

| Check | Count |
|---|---|
| nullPointerRedundantCheck | 51 |
| variableScope | 20 |
| constVariablePointer | 11 |
| unusedStructMember | 3 |
| ctunullpointer | 3 |
| constParameterPointer | 1 |
| arrayIndexOutOfBounds | 1 |
| unreadVariable | 1 |
| knownConditionTrueFalse | 1 |
| arrayIndexOutOfBoundsCond | 1 |

| Severity | Check | File | Line | Message |
|---|---|---|---|---|
| error | arrayIndexOutOfBounds | tasks.c | 3604 | Array '"IDLE"[5]' accessed at index 14, which is out of bounds. |
| warning | arrayIndexOutOfBoundsCond | tasks.c | 5273 | Either the condition '(uxTopPriority)==0' is redundant or the array 'pxReadyTasksLists[5]' is accessed at index 4294967295, which is out of bounds. |
| style | constParameterPointer | stream_buffer.c | 1534 | Parameter 'pxStreamBuffer' can be declared as pointer to const |
| style | constVariablePointer | portable/MemMang/heap_4.c | 574 | Variable 'pxBlock' can be declared as pointer to const |
| style | constVariablePointer | queue.c | 2223 | Variable 'pxQueue' can be declared as pointer to const |
| style | constVariablePointer | queue.c | 2244 | Variable 'pxQueue' can be declared as pointer to const |
| style | constVariablePointer | queue.c | 2643 | Variable 'pxQueue' can be declared as pointer to const |
| style | constVariablePointer | queue.c | 2688 | Variable 'pxQueue' can be declared as pointer to const |
| style | constVariablePointer | stream_buffer.c | 1678 | Variable 'pxStreamBuffer' can be declared as pointer to const |
| style | constVariablePointer | tasks.c | 6498 | Variable 'pucEndOfStack' can be declared as pointer to const |
| style | constVariablePointer | timers.c | 557 | Variable 'pxTimer' can be declared as pointer to const |
| style | constVariablePointer | timers.c | 596 | Variable 'pxTimer' can be declared as pointer to const |
| style | constVariablePointer | timers.c | 685 | Variable 'pxTimer' can be declared as pointer to const |
| style | constVariablePointer | timers.c | 1170 | Variable 'pxTimer' can be declared as pointer to const |
| warning | ctunullpointer | list.c | 150 | Null pointer dereference: pxList |
| warning | ctunullpointer | stream_buffer.c | 1018 | Null pointer dereference: pxStreamBuffer |
| warning | ctunullpointer | stream_buffer.c | 1583 | Null pointer dereference: pxStreamBuffer |
| style | knownConditionTrueFalse | tasks.c | 4858 | Condition 'xConstTickCount==(TickType_t)0U' is always false |
| warning | nullPointerRedundantCheck | queue.c | 708 | Either the condition '(xSemaphore)==0' is redundant or there is possible null pointer dereference: pxSemaphore. |
| warning | nullPointerRedundantCheck | queue.c | 740 | Either the condition '(xSemaphore)==0' is redundant or there is possible null pointer dereference: (struct Queue_t*)xSemaphore. |
| warning | nullPointerRedundantCheck | queue.c | 774 | Either the condition '(pxMutex)==0' is redundant or there is possible null pointer dereference: pxMutex. |
| warning | nullPointerRedundantCheck | queue.c | 833 | Either the condition '(pxMutex)==0' is redundant or there is possible null pointer dereference: pxMutex. |
| warning | nullPointerRedundantCheck | queue.c | 1207 | Either the condition 'pxQueue!=NULL' is redundant or there is possible null pointer dereference: pxQueue. |
| warning | nullPointerRedundantCheck | queue.c | 1383 | Either the condition 'pxQueue!=NULL' is redundant or there is possible null pointer dereference: pxQueue. |
| warning | nullPointerRedundantCheck | queue.c | 1388 | Either the condition 'pxQueue!=NULL' is redundant or there is possible null pointer dereference: pxQueue. |
| warning | nullPointerRedundantCheck | queue.c | 1677 | Either the condition '(pxQueue)==0' is redundant or there is possible null pointer dereference: pxQueue. |
| warning | nullPointerRedundantCheck | queue.c | 2174 | Either the condition 'pxQueue!=NULL' is redundant or there is possible null pointer dereference: pxQueue. |
| warning | nullPointerRedundantCheck | queue.c | 2210 | Either the condition '(xQueue)==0' is redundant or there is possible null pointer dereference: (struct Queue_t*)xQueue. |
| warning | nullPointerRedundantCheck | queue.c | 2231 | Either the condition '(pxQueue)==0' is redundant or there is possible null pointer dereference: pxQueue. |
| warning | nullPointerRedundantCheck | queue.c | 2249 | Either the condition '(pxQueue)==0' is redundant or there is possible null pointer dereference: pxQueue. |
| warning | nullPointerRedundantCheck | queue.c | 2649 | Either the condition '(pxQueue)==0' is redundant or there is possible null pointer dereference: pxQueue. |
| warning | nullPointerRedundantCheck | queue.c | 2694 | Either the condition '(pxQueue)==0' is redundant or there is possible null pointer dereference: pxQueue. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 596 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 628 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 693 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 778 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 779 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 780 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 781 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 785 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 835 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 1018 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 1354 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 1356 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 1426 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 1465 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 1583 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 1584 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 1586 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | stream_buffer.c | 1686 | Either the condition '(pxStreamBuffer)==0' is redundant or there is possible null pointer dereference: pxStreamBuffer. |
| warning | nullPointerRedundantCheck | tasks.c | 2702 | Either the condition 'pxTCB!=NULL' is redundant or there is possible null pointer dereference: pxTCB. |
| warning | nullPointerRedundantCheck | tasks.c | 2752 | Either the condition 'pxTCB!=NULL' is redundant or there is possible null pointer dereference: pxTCB. |
| warning | nullPointerRedundantCheck | tasks.c | 2780 | Either the condition 'pxTCB!=NULL' is redundant or there is possible null pointer dereference: pxTCB. |
| warning | nullPointerRedundantCheck | tasks.c | 2830 | Either the condition 'pxTCB!=NULL' is redundant or there is possible null pointer dereference: pxTCB. |
| warning | nullPointerRedundantCheck | tasks.c | 2880 | Either the condition 'pxTCB!=NULL' is redundant or there is possible null pointer dereference: pxTCB. |
| warning | nullPointerRedundantCheck | tasks.c | 4455 | Either the condition 'pxTCB!=NULL' is redundant or there is possible null pointer dereference: pxTCB. |
| warning | nullPointerRedundantCheck | tasks.c | 4464 | Either the condition 'pxTCB!=NULL' is redundant or there is possible null pointer dereference: pxTCB. |
| warning | nullPointerRedundantCheck | tasks.c | 5481 | Either the condition '(pxEventList)==0' is redundant or there is possible null pointer dereference: pxEventList. |
| warning | nullPointerRedundantCheck | tasks.c | 5672 | Either the condition '(pxTimeOut)==0' is redundant or there is possible null pointer dereference: pxTimeOut. |
| warning | nullPointerRedundantCheck | tasks.c | 5673 | Either the condition '(pxTimeOut)==0' is redundant or there is possible null pointer dereference: pxTimeOut. |
| warning | nullPointerRedundantCheck | tasks.c | 5721 | Either the condition '(pxTicksToWait)==0' is redundant or there is possible null pointer dereference: pxTicksToWait. |
| warning | nullPointerRedundantCheck | tasks.c | 6515 | Either the condition 'pxTCB!=NULL' is redundant or there is possible null pointer dereference: pxTCB. |
| warning | nullPointerRedundantCheck | tasks.c | 8464 | Either the condition 'pxTCB!=NULL' is redundant or there is possible null pointer dereference: pxTCB. |
| warning | nullPointerRedundantCheck | tasks.c | 8506 | Either the condition 'pxTCB!=NULL' is redundant or there is possible null pointer dereference: pxTCB. |
| warning | nullPointerRedundantCheck | tasks.c | 8507 | Either the condition 'pxTCB!=NULL' is redundant or there is possible null pointer dereference: pxTCB. |
| warning | nullPointerRedundantCheck | timers.c | 565 | Either the condition '(xTimer)==0' is redundant or there is possible null pointer dereference: pxTimer. |
| warning | nullPointerRedundantCheck | timers.c | 693 | Either the condition '(xTimer)==0' is redundant or there is possible null pointer dereference: pxTimer. |
| warning | nullPointerRedundantCheck | timers.c | 1207 | Either the condition '(xTimer)==0' is redundant or there is possible null pointer dereference: pxTimer. |
| warning | nullPointerRedundantCheck | timers.c | 1228 | Either the condition '(xTimer)==0' is redundant or there is possible null pointer dereference: pxTimer. |
| style | unreadVariable | event_groups.c | 556 | Variable 'xMatchFound' is assigned a value that is never used. |
| style | unusedStructMember | timers.c | 115 | struct member 'tmrCallbackParameters::pxCallbackFunction' is never used. |
| style | unusedStructMember | timers.c | 116 | struct member 'tmrCallbackParameters::pvParameter1' is never used. |
| style | unusedStructMember | timers.c | 117 | struct member 'tmrCallbackParameters::ulParameter2' is never used. |
| style | variableScope | event_groups.c | 550 | The scope of the variable 'pxListItem' can be reduced. |
| style | variableScope | event_groups.c | 554 | The scope of the variable 'uxControlBits' can be reduced. |
| style | variableScope | event_groups.c | 554 | The scope of the variable 'uxBitsWaitedFor' can be reduced. |
| style | variableScope | event_groups.c | 554 | The scope of the variable 'uxBitsToClear' can be reduced. |
| style | variableScope | portable/MSVC-MingW/port.c | 277 | The scope of the variable 'pxThreadState' can be reduced. |
| style | variableScope | portable/MSVC-MingW/port.c | 396 | The scope of the variable 'ulSwitchRequired' can be reduced. |
| style | variableScope | portable/MSVC-MingW/port.c | 400 | The scope of the variable 'xWinApiResult' can be reduced. |
| style | variableScope | stream_buffer.c | 1019 | The scope of the variable 'xMessageLength' can be reduced. |
| style | variableScope | stream_buffer.c | 1184 | The scope of the variable 'xBytesAvailable' can be reduced. |
| style | variableScope | tasks.c | 2368 | The scope of the variable 'xTicksToWait' can be reduced. |
| style | variableScope | tasks.c | 2368 | The scope of the variable 'xTicksIncrements' can be reduced. |
| style | variableScope | tasks.c | 2417 | The scope of the variable 'xTimeToWake' can be reduced. |
| style | variableScope | tasks.c | 3295 | The scope of the variable 'uxCurrentListLength' can be reduced. |
| style | variableScope | tasks.c | 7802 | The scope of the variable 'xAlreadyYielded' can be reduced. |
| style | variableScope | tasks.c | 7905 | The scope of the variable 'xAlreadyYielded' can be reduced. |
| style | variableScope | timers.c | 572 | The scope of the variable 'pxTimer' can be reduced. |
| style | variableScope | timers.c | 596 | The scope of the variable 'pxTimer' can be reduced. |
| style | variableScope | timers.c | 783 | The scope of the variable 'xTimeNow' can be reduced. |
| style | variableScope | timers.c | 1093 | The scope of the variable 'xNextExpireTime' can be reduced. |
| style | variableScope | timers.c | 1170 | The scope of the variable 'pxTimer' can be reduced. |

## clang-tidy

| Check | Count |
|---|---|
| clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | 14 |
| bugprone-branch-clone | 7 |
| bugprone-signed-bitwise | 5 |
| bugprone-macro-parentheses | 5 |
| clang-analyzer-core.NullDereference | 1 |
| bugprone-narrowing-conversions | 1 |
| bugprone-reserved-identifier | 1 |

| Severity | Check | File | Line | Message |
|---|---|---|---|---|
| warning | bugprone-branch-clone | tasks.c | 2228 | if with identical then and else branches |
| warning | bugprone-branch-clone | tasks.c | 2992 | if with identical then and else branches |
| warning | bugprone-branch-clone | tasks.c | 3214 | if with identical then and else branches |
| warning | bugprone-branch-clone | tasks.c | 6777 | if with identical then and else branches |
| warning | bugprone-branch-clone | tasks.c | 6877 | if with identical then and else branches |
| warning | bugprone-branch-clone | tasks.c | 7010 | if with identical then and else branches |
| warning | bugprone-branch-clone | tasks.c | 8704 | if with identical then and else branches |
| warning | bugprone-macro-parentheses | portable/MemMang/heap_4.c | 82 | macro argument should be enclosed in parentheses |
| warning | bugprone-macro-parentheses | portable/MemMang/heap_4.c | 83 | macro argument should be enclosed in parentheses |
| warning | bugprone-macro-parentheses | portable/MemMang/heap_4.c | 84 | macro argument should be enclosed in parentheses |
| warning | bugprone-macro-parentheses | portable/MSVC-MingW/portmacro.h | 180 | macro argument should be enclosed in parentheses |
| warning | bugprone-macro-parentheses | portable/MSVC-MingW/portmacro.h | 181 | macro argument should be enclosed in parentheses |
| warning | bugprone-narrowing-conversions | portable/MSVC-MingW/port.c | 193 | narrowing conversion from 'TickType_t' (aka 'unsigned int') to signed type 'LONG' (aka 'long') is implementation-defined |
| warning | bugprone-reserved-identifier | FreeRTOSConfig.h (sample, modified copy) | 43 | declaration uses identifier '__FREERTOS_CONFIG_H__', which is a reserved identifier |
| warning | bugprone-signed-bitwise | portable/MemMang/heap_4.c | 192 | use of a signed integer operand with a binary bitwise operator |
| warning | bugprone-signed-bitwise | portable/MemMang/heap_4.c | 195 | use of a signed integer operand with a binary bitwise operator |
| warning | bugprone-signed-bitwise | portable/MemMang/heap_4.c | 272 | use of a signed integer operand with a binary bitwise operator |
| warning | bugprone-signed-bitwise | portable/MemMang/heap_4.c | 279 | use of a signed integer operand with a binary bitwise operator |
| warning | bugprone-signed-bitwise | portable/MemMang/heap_4.c | 465 | use of a signed integer operand with a binary bitwise operator |
| warning | clang-analyzer-core.NullDereference | tasks.c | 4103 | Access to field 'pxIndex' results in a dereference of a null pointer (loaded from variable 'pxList') |
| warning | clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | portable/MemMang/heap_4.c | 448 | Call to function 'memset' is insecure as it does not provide security checks introduced in the C11 standard. Replace with analogous functions that support length arguments or provides boundary checks such as 'memset_s' in case of C11 |
| warning | clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | queue.c | 2425 | Call to function 'memcpy' is insecure as it does not provide security checks introduced in the C11 standard. Replace with analogous functions that support length arguments or provides boundary checks such as 'memcpy_s' in case of C11 |
| warning | clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | queue.c | 2439 | Call to function 'memcpy' is insecure as it does not provide security checks introduced in the C11 standard. Replace with analogous functions that support length arguments or provides boundary checks such as 'memcpy_s' in case of C11 |
| warning | clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | queue.c | 2494 | Call to function 'memcpy' is insecure as it does not provide security checks introduced in the C11 standard. Replace with analogous functions that support length arguments or provides boundary checks such as 'memcpy_s' in case of C11 |
| warning | clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | stream_buffer.c | 596 | Call to function 'memset' is insecure as it does not provide security checks introduced in the C11 standard. Replace with analogous functions that support length arguments or provides boundary checks such as 'memset_s' in case of C11 |
| warning | clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | stream_buffer.c | 1504 | Call to function 'memcpy' is insecure as it does not provide security checks introduced in the C11 standard. Replace with analogous functions that support length arguments or provides boundary checks such as 'memcpy_s' in case of C11 |
| warning | clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | stream_buffer.c | 1512 | Call to function 'memcpy' is insecure as it does not provide security checks introduced in the C11 standard. Replace with analogous functions that support length arguments or provides boundary checks such as 'memcpy_s' in case of C11 |
| warning | clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | stream_buffer.c | 1552 | Call to function 'memcpy' is insecure as it does not provide security checks introduced in the C11 standard. Replace with analogous functions that support length arguments or provides boundary checks such as 'memcpy_s' in case of C11 |
| warning | clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | stream_buffer.c | 1559 | Call to function 'memcpy' is insecure as it does not provide security checks introduced in the C11 standard. Replace with analogous functions that support length arguments or provides boundary checks such as 'memcpy_s' in case of C11 |
| warning | clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | stream_buffer.c | 1645 | Call to function 'memset' is insecure as it does not provide security checks introduced in the C11 standard. Replace with analogous functions that support length arguments or provides boundary checks such as 'memset_s' in case of C11 |
| warning | clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | stream_buffer.c | 1649 | Call to function 'memset' is insecure as it does not provide security checks introduced in the C11 standard. Replace with analogous functions that support length arguments or provides boundary checks such as 'memset_s' in case of C11 |
| warning | clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | tasks.c | 1298 | Call to function 'memset' is insecure as it does not provide security checks introduced in the C11 standard. Replace with analogous functions that support length arguments or provides boundary checks such as 'memset_s' in case of C11 |
| warning | clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | tasks.c | 1693 | Call to function 'memset' is insecure as it does not provide security checks introduced in the C11 standard. Replace with analogous functions that support length arguments or provides boundary checks such as 'memset_s' in case of C11 |
| warning | clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling | tasks.c | 1835 | Call to function 'memset' is insecure as it does not provide security checks introduced in the C11 standard. Replace with analogous functions that support length arguments or provides boundary checks such as 'memset_s' in case of C11 |

