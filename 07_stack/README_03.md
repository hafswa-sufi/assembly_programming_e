# 07_stack/03.asm: call and ret (no stack frame)

## What the program does
It puts 10 in `rax`, calls a function `calculate` that adds 5, and returns.
After the call `rax` is 15. Then the program exits.


## Key ideas
- `call` pushes the **return address** (the next instruction) and jumps.
- `ret` pops that address into **RIP**, so execution continues there.
- This function has no `push rbp`, no `mov rbp, rsp` and no `sub rsp`.
  The only stack changes are from `call` and `ret`.
- The result is passed back in `rax`, not on the stack.

## Step by step (my gdb run)
Start value of RSP, called S, was `0x7fffffffdc50`.

| Instruction | RSP after | RIP after | What happened |
|---|---|---|---|
| start of `_start` | `dc50` (S) | `0x401000` | starting point |
| `mov rax, 10` | `dc50` | next line | rax = 10, stack not touched |
| `call calculate` | `dc48` (S-8) | `0x401015` | return address `0x40100a` saved |
| `add rax, 5` | `dc48` | next line | rax = 15, stack not touched |
| `ret` | `dc50` (S) | `0x40100a` | return address popped into RIP |
| `nop` | `dc50` | after `nop` | back in `_start`, rax = 15 |
| exit syscall | `dc50` | | program ends |

(All addresses start with `0x7fffffffd`, shortened here.)

## Memory view
After `call`, I ran `x/gx $rsp`:
```
0x7fffffffdc48: 0x000000000040100a
```
`0x40100a` is the return address: the address of `after_call`, the
instruction right after the `call`.

## What I observed
1. `call` lowered RSP by 8 and saved `0x40100a` on the stack.
2. Inside `calculate`, RIP was `0x401015`, the function's first instruction.
3. `add rax, 5` did not change RSP. Only `call` and `ret` did.
4. After `ret`, RIP was `0x40100a`, the same value that was on the stack.
   This shows `ret` pops the return address into RIP.
5. RSP ended at S, so the stack was balanced.
6. `rax` was 15 after the call. The result came back in a register.

## Conclusion
`call` saves where to come back to by pushing the return address (RSP down 8).
`ret` pops it back into RIP (RSP up 8). With no frame, those are the only two
changes to the stack, and RSP ends where it started.
