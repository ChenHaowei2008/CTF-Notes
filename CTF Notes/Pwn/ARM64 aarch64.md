Challenges: 
- GreyCTF 2025 Finals rat2libzz 
- YBN CTF 2025 ret3syscall

Run challenges with: `qemu-aarch64 -L . -g 1234 ./chall`

### ARM ROP

| Registers | Purpose             |
| --------- | ------------------- |
| X0-X5     | Argument registers  |
| X8        | Syscall register    |
| X29       | Stack base regsiter |
| X30       | Return address      |
| SP        | Stack pointer       |
| PC        | Program counter     |

| Instruction | Purpose       |
| ----------- | ------------- |
| str         | Store         |
| svc         | Syscall       |
| blr         | Call          |
| ldr         | Load register |
| ldp         | Load pair     |
### General notes

The arm stack layout is very finicky, so just use GDB to always check the layout.
