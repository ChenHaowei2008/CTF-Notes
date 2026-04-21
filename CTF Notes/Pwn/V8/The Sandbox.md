Reading: 
- https://v8.dev/blog/sandbox
- https://theori.io/blog/a-deep-dive-into-v8-sandbox-escape-technique-used-in-in-the-wild-exploit

The point of the v8 sandbox is to replace all pointers with a pointer within the sandbox to point back into the sandbox. This is to prevent attackers from getting access to memory outside of the sandbox. 

For example, an object that may look like this:
```cpp
class JSArrayBuffer: public JSObject {
  private:
    byte* buffer_;
    size_t size_;
};
```

Will be converted into this within the sandbox:
```cpp
class JSArrayBuffer: public JSObject {
  private:
    sandbox_ptr_t buffer_;
    sandbox_size_t size_;
};
```

The `sandbox_ptr_t` is a 40-bit offset from the base of the sandbox, similar to how pointer compression works. The `sandbox_size_t` will become a "sandbox-compatible" size.

If the object requires a buffer that is outside of the sandbox, it will instead become:
```cpp
class JSArrayBuffer: public JSObject {
  private:
    external_ptr_t buffer_;
};
```

This `external_ptr_t` will be an index to a **pointer table** that is located outside of the sandbox, which will then itself contain pointers to external objects.