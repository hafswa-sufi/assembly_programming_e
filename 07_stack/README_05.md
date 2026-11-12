# 07_stack/05.asm: push, pop and little-endian (64-bit)

## What the program does
It loads a big number into `rax` and a small one into `rbx`, pushes both
onto the stack, then pops them back into the same registers. Then it exits.


## Key ideas
- `push` = RSP down by 8, then store. `pop` = read, then RSP up by 8.
- The stack is last in, first out (LIFO).
- Here the pops go back into the **same** registers, in reverse order of
  the pushes, so each register gets its own value back.
- **Little-endian**: a number is stored with its least significant byte
  at the lowest address.

## Step by step (my gdb run)
Start value of RSP, called S, was `0x7fffffffdc50`.

| Instruction | RSP after | What happened |
|---|---|---|
| start of `_start` | `dc50` (S) | starting point |
| `mov rax, 0x1122334455667788` | `dc50` | register only |
| `mov rbx, 0x1234` | `dc50` | register only |
| `push rax` | `dc48` (S-8) | big value stored |
| `push rbx` | `dc40` (S-16) | 0x1234 stored on top |
| `nop` | `dc40` | stack is fullest here |
| `pop rbx` | `dc48` (S-8) | rbx = 0x1234 |
| `pop rax` | `dc50` (S) | rax = 0x1122334455667788 |
| exit syscall | `dc50` | program ends |

(All addresses start with `0x7fffffffd`, shortened here.)

## Memory view
At the `nop` I ran `x/2gx $rsp`:
```
0x7fffffffdc40: 0x0000000000001234      0x1122334455667788
```
- `dc40` (top) = `0x1234`, pushed last.
- `dc48` = `0x1122334455667788`, pushed first.

Then I ran `x/16xb $rsp` to see single bytes:
```
0x7fffffffdc40: 0x34 0x12 0x00 0x00 0x00 0x00 0x00 0x00
0x7fffffffdc48: 0x88 0x77 0x66 0x55 0x44 0x33 0x22 0x11
```
`0x1234` appears as `34 12`, and `0x1122334455667788` appears as
`88 77 66 55 44 33 22 11`. Each number is written backwards byte by byte.
That is little-endian.

## What I observed
1. Each `push` lowered RSP by 8. Each `pop` raised it by 8.
2. The last value pushed (0x1234) was at the lowest address, the top.
3. Pops ran in reverse order, so `rbx` and `rax` got their own values back.
4. In memory the bytes of each number were reversed (little-endian).
5. RSP ended at S, so the stack was balanced.
6. The `mov` instructions did not change RSP.

## Conclusion
Push and pop move RSP by 8 and work last in, first out. Popping in the
reverse order of pushing puts each value back where it came from. Memory
stores the lowest byte of a number first, which is why `0x1234` shows as
`34 12`.
