# 07_stack/06.asm: push, pop and little-endian (32-bit)

## What the program does
It is the 32-bit version of 05.asm. It loads two numbers into `eax` and
`ebx`, pushes both onto the stack, then pops them back into the same
registers. Then it exits.


## Key ideas
- `push` = ESP down by 4, then store. `pop` = read, then ESP up by 4.
- Slots are **4 bytes** in 32-bit mode (8 bytes in 64-bit).
- The stack is last in, first out (LIFO).
- Popping in the reverse order of the pushes gives each register its own
  value back.
- **Little-endian**: the lowest byte of a number is stored first.
- It exits with `int 0x80` instead of `syscall`.

## Step by step (my gdb run)
Start value of ESP, called S, was `0xffffcdb0`.

| Instruction | ESP after | What happened |
|---|---|---|
| start of `_start` | `cdb0` (S) | starting point |
| `mov eax, 0x11223344` | `cdb0` | register only |
| `mov ebx, 0x1234` | `cdb0` | register only |
| `push eax` | `cdac` (S-4) | 0x11223344 stored |
| `push ebx` | `cda8` (S-8) | 0x00001234 stored on top |
| `nop` | `cda8` | stack is fullest here |
| `pop ebx` | `cdac` (S-4) | ebx = 0x1234 |
| `pop eax` | `cdb0` (S) | eax = 0x11223344 |
| exit (`int 0x80`) | `cdb0` | program ends |

(All addresses start with `0xffff`, shortened here.)

## Memory view
At the `nop` I ran `x/2wx $esp`:
```
0xffffcda8:  0x00001234  0x11223344
```
- `cda8` (top) = `0x00001234`, pushed last.
- `cdac` = `0x11223344`, pushed first.

Then I ran `x/8xb $esp` to see single bytes:
```
0xffffcda8:  0x34 0x12 0x00 0x00 0x44 0x33 0x22 0x11
```
`0x1234` appears as `34 12 00 00`, and `0x11223344` appears as
`44 33 22 11`. Each number is written backwards byte by byte.
That is little-endian.

## What I observed
1. Each `push` lowered ESP by 4. Each `pop` raised it by 4.
2. The last value pushed (0x1234) was at the lowest address, the top.
3. Pops ran in reverse order, so `ebx` and `eax` got their own values back.
4. `0x1234` was stored as `34 12 00 00` because a 32-bit slot has 4 bytes.
5. ESP ended at S, so the stack was balanced.
6. Compared with 05.asm, every step was 4 bytes instead of 8.

## Conclusion
Push and pop move ESP by 4 in 32-bit mode and work last in, first out.
Popping in reverse order restores each value to its register. Memory
stores the lowest byte first, so `0x11223344` shows as `44 33 22 11`.
