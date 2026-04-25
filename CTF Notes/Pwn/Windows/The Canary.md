The windows canary works similarly to what we are familiar with. However, an additional security feature that it has over its Linux counterpart, is that it is xor'ed with the stack pointer. This means that in different functions at different points in the stack frame, the canary will have differing values.

Here is how the canary looks like in decompiled code:
```c
// Initialise canary
canary = global_canary ^ (ulonglong)stack_pointer;

// Check canary at the end.
if (global_canary != (canary ^ (ulonglong)stack_pointer)) {
    FUN_1400013c0(canary ^ (ulonglong)stack_pointer);
}
```

