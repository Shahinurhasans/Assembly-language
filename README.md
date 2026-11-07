# Low-Level Computing and Microprocessor Laboratory: Intel 8085 and 8086

This repository contains low-level assembly language routines developed for Intel 8085 and Intel 8086 architectures. Although my primary research focuses on machine learning and trustworthy systems, I maintain this repository as a record of foundational work in computer organization, register-level manipulation, hardware interrupts, and memory-mapped computation.

The primary showcase of this repository is `calc.asm`, an interactive, signed 16-bit arithmetic processor and command-line calculator written for the Intel 8086 processor.

---

## Spotlight Project: calc.asm (Intel 8086 Interactive Calculator)

`calc.asm` is a complete command-line calculator implemented in Intel 8086 assembly. Designed as an MS-DOS `.COM` executable running at memory origin `org 100h`, it operates directly on CPU registers, BIOS interrupt tables, and DOS services without runtime standard libraries.

### 1. Functional Overview
The calculator supports signed four-function integer arithmetic:
* Addition (`+`)
* Subtraction (`-`)
* Multiplication (`*`)
* Integer division (`/`) with remainder reporting

It features full console input parsing, keyboard echo, backspace handling, sign handling, numerical overflow guards, and custom binary-to-decimal string formatting.

### 2. Core Architecture and Register Workflow

#### Memory Layout and State (`org 100h`)
* `num1` (Word): Stores the first 16-bit signed operand.
* `num2` (Word): Stores the second 16-bit signed operand.
* `opr` (Byte): Holds the single-byte operator ASCII code (`+`, `-`, `*`, `/`, or `q` for early exit).
* `ten` (Word, value `10`): Constant multiplier and divisor used in decimal radix conversion.
* `make_minus` (Byte): Flag stored in the code segment (`CS:make_minus`) tracking operand polarity during parsing.

#### Input Acquisition and Parsing (`SCAN_NUM`)
Keyboard input does not rely on high-level buffered input functions. Instead, it processes raw keystrokes character by character:
1. **Keystroke Capture**: Calls BIOS `INT 16h` (service `AH = 00h`) to read keypresses directly from the keyboard buffer.
2. **Teletype Echo**: Invokes BIOS `INT 10h` (service `AH = 0Eh`) to write the character at the current cursor position.
3. **Negative Number Handling**: If the first non-blank character is `-`, the routine sets `CS:make_minus = 1` and waits for subsequent digits.
4. **Digit Accumulation**:
   * Numerical validation checks ASCII range `30h` to `39h` (`0` to `9`). Non-numeric characters trigger backspace correction (`PUTC 8`, `PUTC ' '`, `PUTC 8`).
   * For valid digits, the existing accumulator in register `CX` is scaled by 10 (`MUL CS:ten`).
   * The new digit value (`AL - 30h`) is added to `CX`.
   * High-word overflow is monitored via `DX` after multiplication and the CPU Carry flag (`JC too_big2`). If the value exceeds signed 16-bit capacity, the program automatically rolls back the operation and clears the cursor.
5. **Interactive Backspace**: Pressing Backspace (`ASCII 08h`) divides the current value in `CX` by 10 (`DIV CS:ten`), effectively stripping the least significant digit, and clears the character on screen.
6. **Finalization**: When Enter (`0Dh`) is detected, `CX` is negated via `NEG CX` if `make_minus` is set, returning the final signed value in `CX`.

#### Execution Engine
Once `num1`, `opr`, and `num2` are captured:
* **Addition (`do_plus`)**: Loads `num1` into `AX`, executes `ADD AX, num2`, and passes `AX` to `PRINT_NUM`.
* **Subtraction (`do_minus`)**: Loads `num1` into `AX`, executes `SUB AX, num2`, and outputs `AX`.
* **Multiplication (`do_mult`)**: Uses signed integer multiplication `IMUL num2`. The product resides in `DX:AX`; for small-range calculations, `AX` holds the result.
* **Division (`do_div`)**: Clears `DX` to prepare the 32-bit dividend in `DX:AX`, then performs signed division via `IDIV num2`. The quotient is placed in `AX` and the remainder in `DX`. If `DX` is non-zero, the program prints the integer quotient followed by an approximation warning message (`" and something...."`).

#### Output Formatting (`PRINT_NUM` and `PRINT_NUM_UNS`)
Writing binary numbers back to human-readable ASCII requires custom radix extraction without runtime helper libraries:
1. **Sign Handling**: `PRINT_NUM` checks if the integer in `AX` is negative. If negative, it emits `-` using `INT 10h` and executes `NEG AX` to get the absolute value.
2. **Decimal Decomposition (`PRINT_NUM_UNS`)**:
   * Uses a decaying radix divider initialized at `BX = 10000` (max power of ten within unsigned 16-bit integers).
   * Successively divides `DX:AX` by `BX`. The resulting quotient digit in `AL` is converted to ASCII (`ADD AL, 30h`) and printed.
   * A suppression flag in `CX` skips leading zeros until the first non-zero digit is encountered.
   * `BX` is reduced after each pass by dividing by 10 (`DIV CS:ten`) until all powers of ten have been processed.

---

## Additional Programs in this Repository

Alongside `calc.asm`, this repository contains small laboratory programs written to practice fundamental assembly patterns:

### Intel 8085 Programs
* **`add 16 bit 2 number.asm`**: Basic 8085 accumulator addition. Loads two 8-bit operands from memory addresses `5000H` and `5001H`, computes the sum, handles the carry flag via `INR C`, and stores the 16-bit output across memory addresses `5002H` and `5003H`.

### Intel 8086 Routines
* **`MULTIPLICATION OF 2 8 BIT.asm`**: Reads 8-bit values from data segment offsets `0500H` and `0501H`, executes `MUL BL`, and stores the 16-bit product at offset `0600H`.
* **`mul 16 bit.asm`**: Demonstrates 16-bit multiplication using `MUL BX`, writing the full 32-bit result (`DX:AX`) across destination addresses `10600H` and `10602H`.
* **`div 16 by 8.asm`**: Memory-mapped division routine loading operands through `DS:SI` and writing results to `DS:DI`.
* **`factorial less 8.asm` and `factorial all num.asm`**: Factorial calculation routines demonstrating loop control flow using the `LOOP` instruction and accumulator scaling.
* **`fibonacchii.asm`**: Generates terms of the Fibonacci sequence directly in sequential memory addresses starting from offset `0500H`.

---

## How to Assemble and Run

### For 8086 Programs (including `calc.asm`)
The 8086 programs are targeted for MS-DOS environments. You can run them using `emu8086` or DOSBox with TASM/MASM:

#### Option A: Using emu8086
1. Open `emu8086`.
2. Load `calc.asm`.
3. Click **Compile and Emulate** (F5), then press **Run**.

#### Option B: Using DOSBox with TASM
```text
tasm /m calc.asm
tlink /t calc.obj
calc.com
```

### For 8085 Programs
For `add 16 bit 2 number.asm`, load the source into any standard Intel 8085 simulator (such as **GNUSim8085** or **sim8085**), set input values at memory addresses `5000H` and `5001H`, and execute.
