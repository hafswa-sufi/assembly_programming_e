# ADD and EFLAGS

I ran add1.asm and add2.asm in GDB and checked the flags right after the `add` instruction.

## add1.asm

```
mov al, 120
add al, 10
```

AL = 0x82 (10000010 in binary)

GDB: `eflags 0xa96 [ PF AF SF IF OF ]`

- CF = cleared. 120 + 10 = 130, which fits in 8 bits (max 255), so nothing carried out of bit 7.
- OF = set. 120 and 10 are both positive, but 10000010 has bit 7 = 1, so it reads as negative (-126). The real answer, 130, is bigger than 127, so signed overflow happened.
- SF = set. The top bit (bit 7) of the result is 1.
- ZF = cleared. The result is 0x82, not zero.
- PF = set. The low byte 10000010 has two 1s, which is an even number.
- AF = set. 8 + A = 0x12, so the low 4 bits carried into bit 4.

As unsigned the answer is correct (130), but as signed it is wrong (-126). That's why CF is 0 and OF is 1.

## add2.asm

```
mov ax, 32000
add ax, 500
```

AX = 0x7EF4 (32500)

GDB: `eflags 0x202 [ IF ]`

- CF = cleared. 32500 fits in 16 bits (max 65535).
- OF = cleared. Positive + positive = positive, and 32500 is less than 32767, so no signed overflow.
- SF = cleared. Bit 15 of the result is 0.
- ZF = cleared. The result is not zero.
- PF = cleared. The low byte is 0xF4 = 11110100, which has five 1s, an odd number.
- AF = cleared. 0 + 4 = 4, so no carry out of the low 4 bits.

IF is always on in a normal program, so I ignored it.

