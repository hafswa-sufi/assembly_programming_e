# SUB and EFLAGS

I ran sub1.asm and sub2.asm in GDB and checked the flags right after the `sub` instruction.
For SUB, CF means borrow (it is set when the first number is smaller than the second, as unsigned).

## sub1.asm (8-bit)

```
mov al, 50
sub al, 80
```

AL = 0xE2 (11100010 in binary), which is -30 as signed

GDB: `eflags 0x287 [ CF PF SF IF ]`

- CF = set. 50 is smaller than 80 (unsigned), so it had to borrow.
- OF = cleared. -30 fits in signed 8 bits (-128 to 127). Subtracting two positive numbers can't overflow.
- SF = set. Bit 7 of the result is 1, so it is negative.
- ZF = cleared. The result is not zero.
- PF = set. The low byte 11100010 has four 1s, which is an even number.
- AF = cleared. The low 4 bits were 2 - 0 = 2, so no borrow from bit 4.

As signed the answer is correct (-30), so OF is 0. As unsigned it is wrong (226), so CF is 1.

## sub2.asm (16-bit)

```
mov ax, 1000
sub ax, 2000
```

AX = 0xFC18 (1111110000011000 in binary), which is -1000 as signed

GDB: `eflags 0x287 [ CF PF SF IF ]`

- CF = set. 1000 is smaller than 2000, so it had to borrow.
- OF = cleared. -1000 fits in signed 16 bits (-32768 to 32767).
- SF = set. Bit 15 of the result is 1, so it is negative.
- ZF = cleared. The result is not zero.
- PF = set. PF only looks at the low byte, which is 0x18 = 00011000. That has two 1s, an even number.
- AF = cleared. The low 4 bits were 8 - 0 = 8, so no borrow from bit 4.

