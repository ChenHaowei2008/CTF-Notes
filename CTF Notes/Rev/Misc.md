In gdb, breakpoints and steps work by overwriting the instruction at that address with a trap instruction, usually `int3`. This results in a SIGTRAP when the breakpoint is triggered.

Some challenges may have anti-debugging by setting a signal handler for SIGTRAP. Self-modifying challenges that read and write to its own code can also be affected by the trap instructions that gdb inserts. 

To circumvent this, use `hbreak`, hardware break, which doesn't use trap instructions or call SIGTRAP. Instead, it uses debug registers. However, you can only have a very limited amount of hardware breakpoints at a time.