Challenges: 
- pwn-college quarterly quiz

Debugging set-up:
Use the `--allow-natives-syntax` flag when debugging d8 (debug build of v8).

This lets you run the following syntax:

```js
%DebugPrint(obj)
%SystemBreak()
```