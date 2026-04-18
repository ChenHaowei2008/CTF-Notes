Challenges: b01lersc.tf micromicromicropython

I've only ever done FSOP on musl 1.2.5, so maybe older versions are different.

Most of my knowledge comes from [this](https://nouxia7.github.io/nouxia-notes/posts/musl-fsop-arkavidia-2025/).

FSOP on musl libc is very simple. How it is triggered is usually upon exit. The code which can be see [here](https://elixir.bootlin.com/musl/v1.2.5/source/src/exit/exit.c#L27):
```c
_Noreturn void exit(int code)
{
	__funcs_on_exit();
	__libc_exit_fini();
	__stdio_exit();
	_Exit(code);
}
```

We will focus on `__stdio_exit()`, source code [here](https://elixir.bootlin.com/musl/v1.2.5/source/src/stdio/__stdio_exit.c#L16):
```c

static void close_file(FILE *f)
{
	if (!f) return;
	FFINALLOCK(f);
	if (f->wpos != f->wbase) f->write(f, 0, 0);
	if (f->rpos != f->rend) f->seek(f, f->rpos-f->rend, SEEK_CUR);
}

void __stdio_exit(void)
{
	FILE *f;
	for (f=*__ofl_lock(); f; f=f->next) close_file(f);
	close_file(__stdin_used);
	close_file(__stdout_used);
	close_file(__stderr_used);
}
```

As seen, the requirements for triggering FSOP is quite simple.
1) `f` must be `/bin/sh` or whatever we wan to run
2) `f->wpos` (+0x28) != `f->wbase` (+0x38)
3) `f->write` (+0x48) must be `system` or the function we want to call

The struct for a `FILE` struct can be seen [here](https://elixir.bootlin.com/musl/v1.2.5/source/src/internal/stdio_impl.h#L21):

```c
struct _IO_FILE {
	unsigned flags;
	unsigned char *rpos, *rend;
	int (*close)(FILE *);
	unsigned char *wend, *wpos;
	unsigned char *mustbezero_1;
	unsigned char *wbase;
	size_t (*read)(FILE *, unsigned char *, size_t);
	size_t (*write)(FILE *, const unsigned char *, size_t);
	off_t (*seek)(FILE *, off_t, int);
	unsigned char *buf;
	size_t buf_size;
	FILE *prev, *next;
	int fd;
	int pipe_pid;
	long lockcount;
	int mode;
	volatile int lock;
	int lbf;
	void *cookie;
	off_t off;
	char *getln_buf;
	void *mustbezero_2;
	unsigned char *shend;
	off_t shlim, shcnt;
	FILE *prev_locked, *next_locked;
	struct __locale_struct *locale;
};
```

That is all.