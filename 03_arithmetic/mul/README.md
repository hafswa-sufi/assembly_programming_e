# MUL and EFLAGS

I ran mul1.asm and mul2.asm in GDB and checked the flags right after the `mul` instruction.
Both are unsigned `mul`. After `mul`, only CF and OF mean something. They are set when the top half of the result (AH for 8-bit, DX for 16-bit) is not zero, which means the result
did not fit in the lower half. They are cleared when the top half is zero.
SF, ZF, AF and PF are undefined after `mul` 

## mul1.asm (8-bit)

```
mov al, 25
mul byte [num2]    ; num2 = 10
```

AX = 0x00FA (250), AH = 0

GDB: `eflags 0x202 [ IF ]`

- CF = cleared. 250 fits in AL, so AH is 0.
- OF = cleared. Same reason, AH is 0.
- SF, ZF, AF, PF = undefined. GDB showed them as cleared, but I can't rely on that.
  For example the result 0xFA has bit 7 = 1 and an even number of 1s in the low byte,
  so with `add` SF and PF would have been set. Here they are not, which shows `mul`
  does not calculate them.

## mul2.asm (16-bit)

```
mov ax, 3000
mul word [num2]    ; num2 = 200
```

DX:AX = 0x0009:0x27C0 (600000), so DX = 9

GDB: `eflags 0xa03 [ CF IF OF ]`

- CF = set. 600000 is too big for 16 bits, so part of it went into DX (DX = 9).
- OF = set. Same reason, DX is not 0.
- SF, ZF, AF, PF = undefined. GDB showed them as cleared, but I can't rely on that.

## Note

The only difference between the two programs is where the answer ended up. In mul1 it
stayed in AL/AX with the top half at 0, so CF and OF were 0. In mul2 it spilled into DX,
so CF and OF were 1. 