# 07_stack/01.asm: How the stack works

## What the program does
It puts two numbers (100 and 200) on the stack with `push`, then takes them
off with `pop` into two different registers. Then it exits.


## Key ideas
- The **stack** is a pile of 8-byte slots in memory (64-bit).
- **RSP** is a register that holds the address of the top of the pile.
- `push` = RSP goes **down** by 8, then the value is stored there.
- `pop` = the value at RSP is read into a register, then RSP goes **up** by 8.
- The stack grows **downward** (toward smaller addresses).
- **LIFO**: last in, first out.

## Step by step (my gdb run)
Start value of RSP, called S, was `0x7fffffffdc60`.
The stack is not empty at the start. Linux already put the program's
arguments above S. My program only uses the space below S.

| Instruction | RSP after | What happened |
|---|---|---|
| start of `_start` | `0x7fffffffdc60` (S) | starting point |
| `mov rax, 100` | S | register only, stack not touched |
| `mov rbx, 200` | S | register only, stack not touched |
| `push rax` | `0x7fffffffdc58` (S-8) | 100 stored on the stack |
| `push rbx` | `0x7fffffffdc50` (S-16) | 200 stored on top of 100 |
| `nop` | `0x7fffffffdc50` | does nothing, stack is fullest here |
| `pop rcx` | `0x7fffffffdc58` (S-8) | top value 200 goes into `rcx` |
| `pop rdx` | `0x7fffffffdc60` (S) | next value 100 goes into `rdx` |
| exit syscall | S | program ends |

## Memory view
At the `nop` I ran `x/2gx $rsp`:
```
0x7fffffffdc50: 0x00000000000000c8      0x0000000000000064
```
- `0x...dc50` (where RSP points, the top) holds `0xc8` = 200, pushed last.
- `0x...dc58` holds `0x64` = 100, pushed first.

So 100 sits at the higher address and 200 at the lower one,
because the stack grows downward.

## What I observed
1. Each `push` lowered RSP by 8 (`dc60` -> `dc58` -> `dc50`).
2. Each `pop` raised RSP by 8 (`dc50` -> `dc58` -> `dc60`).
3. Values came out in reverse order (LIFO): `rcx = 200`, `rdx = 100`.
4. RSP ended at the same value it started with, so pushes and pops were balanced.
5. The `mov` instructions did not change RSP. Only `push` and `pop` do.
6. The addresses can differ on another run or machine, but the pattern is the same.

## Conclusion
`push` stores a value and moves RSP down by 8. `pop` reads a value and moves
RSP up by 8. Because it is last in, first out, the second pop gets what the
first push stored. Equal numbers of pushes and pops leave RSP where it began.
