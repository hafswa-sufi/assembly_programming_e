# 07_stack/02.asm: Stack frame of a function (64-bit)

## What the program does
`_start` calls a function called `work`. The function makes its own
workspace on the stack, stores two local variables (100 and 200), reads
them back into `rax` and `rbx`, cleans up, and returns. Then the program exits.


## Key ideas
- `call work` pushes the **return address** (where to come back to), then jumps.
- **RBP** marks the start of the function's workspace (the **stack frame**).
  It stays fixed while RSP moves.
- `sub rsp, 16` reserves 16 bytes for local variables in one step.
- `mov rsp, rbp` throws the locals away.
- `ret` pops the return address and jumps back.

## Step by step (my gdb run)
Start value of RSP, called S, was `0x7fffffffdc50`.

| Instruction | RSP after | RBP after | What happened |
|---|---|---|---|
| start of `_start` | `dc50` (S) | 0 | starting point |
| `call work` | `dc48` (S-8) | 0 | return address `0x401005` saved |
| `push rbp` | `dc40` (S-16) | 0 | old rbp (0) saved |
| `mov rbp, rsp` | `dc40` | `dc40` | frame starts here |
| `sub rsp, 16` | `dc30` (S-32) | `dc40` | 16 bytes reserved |
| `mov [rbp-8], 100` | `dc30` | `dc40` | 100 stored at `dc38` |
| `mov [rbp-16], 200` | `dc30` | `dc40` | 200 stored at `dc30` |
| `mov rax,[rbp-8]` | `dc30` | `dc40` | rax = 100 |
| `mov rbx,[rbp-16]` | `dc30` | `dc40` | rbx = 200 |
| `nop` | `dc30` | `dc40` | stack is fullest here |
| `mov rsp, rbp` | `dc40` (S-16) | `dc40` | locals thrown away |
| `pop rbp` | `dc48` (S-8) | 0 | old rbp restored |
| `ret` | `dc50` (S) | 0 | returns to `_start` |

(All addresses start with `0x7fffffffd`, shortened here.)

## Memory view
At the `nop` I ran `x/4gx $rsp`:
```
0x7fffffffdc30: 0x00000000000000c8      0x0000000000000064
0x7fffffffdc40: 0x0000000000000000      0x0000000000401005
```
From the top (RSP) going up:
- `dc30` = 200 (local 2, at rbp-16)
- `dc38` = 100 (local 1, at rbp-8)
- `dc40` = 0 (saved old rbp)
- `dc48` = `0x401005` (return address)

The locals sit below RBP. The saved rbp and return address sit at and above it.

## What I observed
1. `call` lowered RSP by 8 and saved the return address `0x401005`,
   which is the instruction after the `call`.
2. `push rbp` lowered RSP by 8 again. `mov rbp, rsp` made RBP equal RSP.
3. `sub rsp, 16` lowered RSP by 16 without any push.
4. The locals were found using RBP (`[rbp-8]`, `[rbp-16]`), not RSP.
5. `mov rsp, rbp` raised RSP by 16 and threw the locals away.
6. `pop rbp` restored RBP to 0, and `ret` popped the return address.
7. RSP ended at S, the same as the start. The stack was balanced.

## Conclusion
A function saves the return address (`call`), saves the old RBP
(`push rbp`), sets a new RBP, and reserves space with `sub rsp`.
To leave, it does the reverse: `mov rsp, rbp`, `pop rbp`, `ret`.
Everything it added to the stack is removed, so RSP returns to S.
