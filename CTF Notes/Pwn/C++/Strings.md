Reading: 
- https://github.com/ir0nstone/cybersec-notes/blob/master/rev/strings-in-c%2B%2B.md

Strings contain 3 fields: 
1) Pointer to the buffer
2) Logical size of the string
3) Allocated memory to the buffer

However, due to small string optimisation, if the string being stored can fit within the structure itself, it doesn't store the `allocated_len`, and just stores the strong constant there.

Thus, the struct looks like this (code from [here](https://github.com/ir0nstone/cybersec-notes/blob/master/rev/strings-in-c%2B%2B.md), `local_buf` size modified to reflect gcc):

```c
class std::string
{
    char* buf;
    size_t len;
    
    // union is used to store different data types in the same memory location
    // this saves space in case only one of them is necessary 
    union
    {
        size_t allocated_len;
        char local_buf[24];
    };
};
```

When a new string is assigned to a string variable, but it's length exceeds the string object's capacity, it will reallocate a new backing for the string. However, if the new string is shorter than the string object's capacity, it will simply put the new string in the same allocated address.

A string initialises with this:
```
0x55cd1182b2f0: 0x000055cd1182b300      0x0000000000000000  
0x55cd1182b300: 0x0000000000000000      0x0000000000000000
```

As seen, `string->buf` points to `string->local_buf`, and `string->len` is 0.

