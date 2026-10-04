# DIV and EFLAGS

I ran div1.asm and div2.asm in GDB and checked the flags before and after the `div` instruction.
After `div`, all six flags (CF, OF, SF, ZF, AF, PF) are undefined .

## div1.asm (8-bit divisor)

```
mov ax, 100
mov bl, 7
div bl        ; AX / BL
```

For an 8-bit divisor, AX is divided by BL. The quotient goes to AL and the remainder to AH.

Result: AL = 0x0E (14), AH = 0x02 (2). Check: 14 * 7 + 2 = 100.

GDB before div: `eflags 0x202 [ IF ]`
GDB after div:  `eflags 0x212 [ AF IF ]`

The flags changed: AF turned on. But AF is undefined after `div`, so this is just what
my CPU left in it. I can't say it means a carry from bit 3, because `div` does not
calculate AF. CF, OF, SF, ZF and PF stayed cleared, but they are also undefined, so
I can't rely on that either.

## div2.asm (16-bit divisor)

```
mov ax, 50000
mov dx, 0
mov bx, 300
div bx        ; DX:AX / BX
```

For a 16-bit divisor, DX:AX is divided by BX. The quotient goes to AX and the remainder to DX.
I set DX to 0 first because DX is the top half of the number being divided.

Result: AX = 0x00A6 (166), DX = 0x00C8 (200). Check: 166 * 300 + 200 = 50000.

GDB before div: `eflags 0x202 [ IF ]`
GDB after div:  `eflags 0x212 [ AF IF ]`

Same as div1: AF turned on, and since it is undefined after `div` I can't explain it.

