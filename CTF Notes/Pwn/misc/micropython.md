Challenges: b01lersc.tf micromicromicropython

All integers are tagged: `(num << 1) | 1`.

Here are some examples of how certain micropython objects look like:

**List:**
```c
typedef struct _mp_obj_list_t {
    mp_obj_base_t base;   // type ptr
    size_t alloc;         // allocated size
    size_t len;           // current length
    mp_obj_t *items;      // pointer to element array
} mp_obj_list_t;
```

**Tuple:**
```c
typedef struct _mp_obj_tuple_t {
    mp_obj_base_t base;   // type ptr
    size_t len;
    mp_obj_t items[];     // inline array
} mp_obj_tuple_t;
```

**Bytes:**
```c
typedef struct _mp_obj_str_t {
    mp_obj_base_t base;   // type ptr
    size_t hash;
    size_t len;
    const byte *data;
} mp_obj_str_t; 
```

**Dict:**
```c
typedef struct _mp_obj_dict_t {
    mp_obj_base_t base;
    mp_map_t map;
} mp_obj_dict_t;

typedef struct _mp_map_t {
    size_t all_keys_are_qstrs : 1;
    size_t is_fixed : 1;    // if set, table is fixed/read-only
    size_t is_ordered : 1;  // if set, table is an ordered array, not hash map
    size_t used : (8 * sizeof(size_t) - 3);
    size_t alloc;
    mp_map_elem_t *table;
} mp_map_t;

typedef struct _mp_map_elem_t {
    mp_obj_t key;
    mp_obj_t value;
} mp_map_elem_t;
```

For a dict, the first four fields are packed into one `size_t` unit. Therefore, a more simplified view of the dict would be:
```c
typedef struct _mp_obj_dict_t {
    mp_obj_base_t base;
    size_t flags;
    size_t alloc;
    mp_map_elem_t *table;
} mp_obj_dict_t;
```

With a type confusion primitive, notice that the first element of a tuple perfect lines up with the `alloc` field for a dict, as well as the `len` field of a list. This is extremely useful in getting fakeobj or arbread/arbwrite primitives, as the tuple writes into inline memory, and can forge fields in these objects.