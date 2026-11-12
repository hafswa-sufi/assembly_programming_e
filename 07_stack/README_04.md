# 07_stack/04.asm: Stack frame of a function (32-bit)

## What the program does
It is the 32-bit version of 02.asm. `_start` calls `work`. The function
makes a workspace on the stack, stores two locals (100 and 200), reads
them into `eax` and `ebx`, cleans up and returns. Then the program exits.


## Key ideas
- Same idea as 02.asm, but slots are **4 bytes**, not 8.
- Registers are `esp` and `ebp` (not `rsp`/`rbp`).
- `call` pushes the return address. `ret` pops it.
- `ebp` marks the start of the frame. `sub esp, 8` reserves 8 bytes.
- It exits with `int 0x80` instead of `syscall`.

## Step by step (my gdb run)
Start value of ESP, called S, was `0xffffcdb0`.

| Instruction | ESP after | EBP after | What happened |
|---|---|---|---|
| start of `_start` | `cdb0` (S) | 0 | starting point |
| `call work` | `cdac` (S-4) | 0 | return address `0x08049005` saved |
| `push ebp` | `cda8` (S-8) | 0 | old ebp (0) saved |
| `mov ebp, esp` | `cda8` | `cda8` | frame starts here |
| `sub esp, 8` | `cda0` (S-16) | `cda8` | 8 bytes reserved |
| `mov [ebp-4], 100` | `cda0` | `cda8` | 100 stored at `cda4` |
| `mov [ebp-8], 200` | `cda0` | `cda8` | 200 stored at `cda0` |
| `mov eax,[ebp-4]` | `cda0` | `cda8` | eax = 100 |
| `mov ebx,[ebp-8]` | `cda0` | `cda8` | ebx = 200 |
| `nop` | `cda0` | `cda8` | stack is fullest here |
| `mov esp, ebp` | `cda8` (S-8) | `cda8` | locals thrown away |
| `pop ebp` | `cdac` (S-4) | 0 | old ebp restored |
| `ret` | `cdb0` (S) | 0 | back in `_start` |

(All addresses start with `0xffff`, shortened here.)

## Memory view
At the `nop` I ran `x/4wx $esp`:
```
0xffffcda0:  0x000000c8  0x00000064  0x00000000  0x08049005
```
From the top (ESP) going up, each slot is 4 bytes:
- `cda0` = 200 (local 2, at ebp-8)
- `cda4` = 100 (local 1, at ebp-4)
- `cda8` = 0 (saved old ebp)
- `cdac` = `0x08049005` (return address)

## What I observed
1. `call` lowered ESP by 4 and saved `0x08049005`, the instruction after it.
2. `push ebp` lowered ESP by 4 again. `mov ebp, esp` made EBP equal ESP.
3. `sub esp, 8` reserved 8 bytes without any push.
4. Locals were reached with EBP (`[ebp-4]`, `[ebp-8]`).
5. `mov esp, ebp` threw the locals away. `pop ebp` restored EBP to 0.
6. `ret` popped the return address, so ESP ended at S. The stack was balanced.
7. Compared with 02.asm, every step was 4 bytes instead of 8.

## Conclusion
A 32-bit function builds a frame the same way as a 64-bit one:
`call`, `push ebp`, `mov ebp, esp`, `sub esp, N`. It leaves with
`mov esp, ebp`, `pop ebp`, `ret`. Everything added is removed, so ESP
returns to S. The only difference is the slot size: 4 bytes.
