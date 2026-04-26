Assume that we have arbread/arbwrite primitives. Now, we want to get RCE. On Linux-based pwn, the standard way to achieve this would be to call `system("/bin/sh")`. However, on Windows, we don't have the `system` function, and either have to:

1) Rely on Windows API
2) Leak libc base and call `system("/bin/sh")`

Method 1 is long and tedious, as Windows API functions require many arguments to be setup. 

Method 2 is only possible with challenges that are run with Wine. 

In programs run with Wine, the loaded dlls, such as `ucrtbase.dll` or `kernel32.dll` have a **constant address**. They are not affected by ASLR at all. However, as wine requires libc and other libraries to run, these linux shared objects are imported as well. Unlike their Windows counterparts, they are **ASLR randomised**.

We can see this in action by triggering a page fault on a wine process:
```
Modules:  
Module  Address                                 Debug info      Name (11 modules)  
PE             140000000-       140006000       Deferred        chall  
PE-Wine     6ffffe0c0000-    6ffffe0dd000       Deferred        vcruntime140  
PE-Wine     6ffffe9c0000-    6ffffedfb000       Deferred        ucrtbase  
PE-Wine     6fffff3a0000-    6fffff9ee000       Deferred        kernelbase  
PE-Wine     6fffffa00000-    6fffffbc9000       Deferred        kernel32  
PE-Wine     6fffffbe0000-    6ffffffe9000       Deferred        ntdll  
ELF         7f007a400000-    7f007a613000       Deferred        libc.so.6  
ELF         7f007a658000-    7f007a685000       Deferred        libgcc_s.so.1  
ELF         7f007a685000-    7f007a747000       Dwarf-4         ntdll.so  
ELF         7f007a792000-    7f007a7cf000       Deferred        ld-linux-x86-64.so.2  
ELF         7f007a7cf000-    7f007a7d4000       Deferred        <wine-loader>
```

This is very useful to us, as it means that any gadgets that we can find in the dlls can be available to us at all times. Now, we just have to figure out a way to leak libc base.

We can do this through the **SharedUserData page**. On Windows, this is a fixed, user-mapped page used by the kernel to expose things like time, tick count, etc. Thus, Wine needs to emulate this region. In order to do so, it contains pointers to `ntdll.so`, which allows us to leak ASLR!

The address to look out for specifically, is `0x7ffe1000`.

Let's examine this region in `winedbg`. We can run `winedbg --gdb ./chall.exe`.

```
pwndbg> x/10gx 0x7ffe1000  
0x7ffe1000:     0x00007f9aa011ca44      0x0000000000000000  
0x7ffe1010:     0x0000000000000000      0x0000000000000000
```

As shown above, this address contains a pointer that allows us to leak ASLR.

From there, it is trivial to do the usual `ret2libc` chain. 

I am actually surprised that calling `system()` from within Wine doesn't cause undefined behaviour, as I'd expect Wine to treat the syscall as one that came from the Windows program, and translate it into something irrelevant. I do more research on how Wine works under the hood.