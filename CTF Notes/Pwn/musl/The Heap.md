### Malloc/Allocating a Chunk

There are two major versions of malloc in musl.

In musl versions 1.2.1 and above, it will use the mallocng allocator. I haven't encountered a musl heap challenge that uses this version yet. (Use `nc-*` commands in pwndbg to analyse the heap).

In versions below 1.2.1, it uses the old dlmalloc-like allocator.

The code for this allocator is very simple (taken from [1.1.16](https://elixir.bootlin.com/musl/v1.1.16/source/src/malloc/malloc.c#L320)):

```c
void *malloc(size_t n)
{
	struct chunk *c;
	int i, j;

	if (adjust_size(&n) < 0) return 0;

	if (n > MMAP_THRESHOLD) {
		size_t len = n + OVERHEAD + PAGE_SIZE - 1 & -PAGE_SIZE;
		char *base = __mmap(0, len, PROT_READ|PROT_WRITE,
			MAP_PRIVATE|MAP_ANONYMOUS, -1, 0);
		if (base == (void *)-1) return 0;
		c = (void *)(base + SIZE_ALIGN - OVERHEAD);
		c->csize = len - (SIZE_ALIGN - OVERHEAD);
		c->psize = SIZE_ALIGN - OVERHEAD;
		return CHUNK_TO_MEM(c);
	}

	i = bin_index_up(n);
	for (;;) {
		uint64_t mask = mal.binmap & -(1ULL<<i);
		if (!mask) {
			c = expand_heap(n);
			if (!c) return 0;
			if (alloc_rev(c)) {
				struct chunk *x = c;
				c = PREV_CHUNK(c);
				NEXT_CHUNK(x)->psize = c->csize =
					x->csize + CHUNK_SIZE(c);
			}
			break;
		}
		j = first_set(mask);
		lock_bin(j);
		c = mal.bins[j].head;
		if (c != BIN_TO_CHUNK(j)) {
			if (!pretrim(c, n, i, j)) unbin(c, j);
			unlock_bin(j);
			break;
		}
		unlock_bin(j);
	}

	/* Now patch up in case we over-allocated */
	trim(c, n);

	return CHUNK_TO_MEM(c);
}
```

The struct for `chunk` is as shown:

```c
struct chunk {
	size_t psize, csize;
	struct chunk *next, *prev;
};
```

At the start, the size is adjusted to account for metadata. Afterwards, there are two main paths.

In the first path, malloc checks if the size requested is very large. Specifically, it checks if the size is more than `MMAP_THRESHOLD`, which is a macro defined as such:

```c
#define SIZE_ALIGN (4*sizeof(size_t))
#define SIZE_MASK (-SIZE_ALIGN)
#define OVERHEAD (2*sizeof(size_t))
#define MMAP_THRESHOLD (0x1c00*SIZE_ALIGN)
```

This is `0xe0000` on 32-bit and `0x1c0000` on 64-bit.

It then uses `mmap` to allocate space for this chunk.

In the other path, malloc computes the smallest acceptable bin for our size, by searching `mal.binmap` for a non-empty bin at or above that size. If it doesn't exist, the heap is grown with `brk`, or `mmap` if it fails. If it does exist, malloc takes a chunk from the smallest suitable non-empty bin. If the chosen chunk is larger than needed, it splits off the tail with `trin(c, n)`.

These bins are **doubly linked-lists** that are unlinked when a chunk is requested, and linked when a chunk is freed.