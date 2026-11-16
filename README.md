# koiafk OS: Fantasy Console & CPU Emulator

A fantasy console built from scratch in **one HTML file** with no libraries and no build step. It has a custom CPU, a two-pass assembler, a 64 KB memory map, a memory-mapped 64x64 display, and a Windows 98 / Frutiger Aero desktop around it.

Open `index.html` in any modern browser. That's the whole install.

**[Launch the Emulator](https://koiafk.github.io/fantasy-console/)**

## What's in it

- **CPU:** 8-bit data path (A, X, Y) with 16-bit addresses (PC, SP), a zero flag, and 19 instructions.
- **Assembler:** two passes (syntax and label addresses, then label resolution). Strict: it rejects bad syntax with the line number, wipes program memory, and refuses to run until the error is fixed.
- **Memory:** 64 KB. VRAM is mapped at `0x8000-0x8FFF` (64x64 pixels, one byte each).
- **Display:** only writes inside the VRAM range touch the canvas.
- **Debugger:** step, run/pause, breakpoints (click a line number), live registers, disassembly of the next three instructions, and a clickable hex memory viewer.
- **Clock:** type any speed from 1 Hz to 1,000,000 Hz. The UI shows instructions executed and the speed actually achieved.
- **Demos:** Full Screen Fill, Diagonal Stripes, Checkerboard, Border Frame, Clear Screen.
- **Desktop:** draggable windows, a Documentation folder with a `guide.md`, and jokes when you try to close the window.

## Instruction set

| Opcode | Mnemonic | Operand | Description |
|---|---|---|---|
| 0x01 | LDA | `#val` | Load immediate into A |
| 0x02 | LDX | `#val` | Load immediate into X |
| 0x03 | STA | `addr,X` | Store A at `addr + X` |
| 0x04 | ADD | `#val` | A = A + val (sets ZF) |
| 0x05 | SUB | `#val` | A = A - val (sets ZF) |
| 0x06 | INC | `X` | Increment X (sets ZF) |
| 0x07 | CMP | `#val` | ZF = (A == val) |
| 0x08 | JMP | `addr` | Jump |
| 0x09 | JZ | `addr` | Jump if ZF = 1 |
| 0x0A | PUSH | `A` | Push A |
| 0x0B | POP | `A` | Pop into A |
| 0x0C | CALL | `addr` | Push return address, jump |
| 0x0D | RET | | Return |
| 0x0E | HLT | | Halt |
| 0x0F | TAX | | X = A (sets ZF) |
| 0x10 | TXA | | A = X (sets ZF) |
| 0x11 | INC | `Y` | Increment Y (sets ZF) |
| 0x12 | TAY | | Y = A (sets ZF) |
| 0x13 | TYA | | A = Y (sets ZF) |

Immediates must start with `#` and fit in 0-255. Numbers can be decimal, `0x1F`, or `$1F`. `LDA` and `LDX` do not change the zero flag.

## Memory map

| Range | Use |
|---|---|
| `0x0000-0x7FFF` | Code and general RAM |
| `0x8000-0x8FFF` | VRAM, pixel address = `0x8000 + row*64 + column` (0 = off, nonzero = on) |
| `0xFF00` | Keyboard port (WASD / arrows) |
| `0xFF00-0xFFFF` | Stack, grows down from `0xFFFF` |

## Example

```asm
; light the top row
LDA #1
LDX #0
loop:
STA 0x8000,X
INC X
TXA
CMP #64
JZ done
LDA #1          ; TXA overwrote A, so reload it
JMP loop
done:
HLT
```

## Known limitations

- **The keyboard port is write-only from the program's point of view.** The emulator updates `0xFF00` on key presses, but the ISA has no load-from-memory instruction, so programs can't read it yet. A `LDA addr` instruction is the obvious next step.
- STA is the only memory-write instruction and it stores A with an 8-bit index, so drawing is page-based (see the demos).
- The speed you type is a target. If your machine can't keep up, the "Actual" readout shows what it really achieved.

## Tests

The emulator was checked headlessly (jsdom with a recording canvas): every demo is run to `HLT` and the framebuffer compared pixel by pixel, VRAM bounds are verified, every assembler error path is checked to block execution, and the HLT-then-Run-again restart is verified.

## Author

koiafk. Single-file, no dependencies.
