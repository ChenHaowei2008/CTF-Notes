Challenges:
- Dreamhack tiny-backdoor

This can be found in the .fini_array section of a program. You can find this section using gdb's `info file` command.

It defines what functions will be run on program termination.

It is an array of pointers, that will be run backwards from the end of the fini_array to the start, with a pointer that keeps track of which fini_array element should be run. It seems that this pointer is persistent even if `_start` is called again, implying it must be stored in some global counter.

```c
/* This function should not be used anymore.  We run the executable's
   destructor now just like any other.  We cannot remove the function,
   though.  */
void
__libc_csu_fini (void)
{
#ifndef LIBC_NONSHARED
  size_t i = __fini_array_end - __fini_array_start;
  while (i-- > 0)
    (*__fini_array_start [i]) ();

# ifndef NO_INITFINI
  _fini ();
# endif
#endif
}
```

This code likely isn't the one being called to run the fini_array functions, but it demonstrates how the pointers are called.

Apparently, the fini_array is protected when FULL RELRO is enabled. I have yet to test this though.