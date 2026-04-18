Challenges: b01lersc.tf micromicromicropython

Even-numbered integers are tagged: `(num << 1) | 1`. Odd integers are directly stored in memory.

Micropython objects look something like this:
```
8 bytes ptr to type
8 bytes allocated mem
8 bytes len
8 bytes ptr (for dicts, lists, bytes)
or just data
```

