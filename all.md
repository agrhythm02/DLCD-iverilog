# DLCD Lab Exam Complete Preparation & Revision Guide (Labs 0, 2–9)
**Course:** Digital Logic & Computer Design (Verilog / Icarus Verilog)  
**Target:** 20-minute MCQ Lab Exam Master Notes

---

## 📑 Table of Contents
1. [3-Minute High-Yield Cheat Sheet](#1-3-minute-high-yield-cheat-sheet)
2. [Sized Number Literals Syntax](#2-sized-number-literals-syntax)
3. [LAB 0: Icarus Verilog Toolkit & Testbench Basics](#3-lab-0-icarus-verilog-toolkit--testbench-basics)
4. [LAB 2: Logic Gates and Structural Circuits](#4-lab-2-logic-gates-and-structural-circuits)
5. [LAB 3: MUX, Decoder, and Priority Encoder](#5-lab-3-mux-decoder-and-priority-encoder)
   - *Special Focus: How to solve 8-to-3 Priority Encoder in 5 seconds*
   - *Special Focus: Input `00000000` vs `00000001` (`VALID` bit)*
6. [LAB 4: Data-Flow Modelling](#6-lab-4-data-flow-modelling)
   - *Special Focus: `~a` (Bitwise NOT) vs `!a` (Logical NOT)*
7. [LAB 5: Sequential Circuits (Latches & Flip-Flops)](#7-lab-5-sequential-circuits-latches--flip-flops)
8. [LAB 6: Full Adder & 8-Bit Ripple-Carry Adder (RCA)](#8-lab-6-full-adder--8-bit-ripple-carry-adder-rca)
   - *Special Focus: Decimal-to-Binary Mental Math for 8-bit Addition*
9. [LAB 7: 16-Bit Arithmetic Logic Unit (ALU)](#9-lab-7-16-bit-arithmetic-logic-unit-alu)
   - *Special Focus: Carry vs Overflow*
10. [LAB 8: 32-Bit Comparators](#10-lab-8-32-bit-comparators)
    - *Special Focus: Signed vs Unsigned & Case Equality (`===`)*
11. [LAB 9: Register File (8 × 16-Bit)](#11-lab-9-register-file-8--16-bit)
    - *Special Focus: Register 0 (`R0`) & Flip-Flop Counts*

---

## 1. 3-Minute High-Yield Cheat Sheet

| Topic | Golden Rules to Memorize |
| :--- | :--- |
| **0. Toolchain** | • **Compile**: `iverilog -o <outfile> file1.v file2.v`<br>• **Simulate/Run**: `vvp <outfile>`<br>• **Waveform**: `gtkwave <dump.vcd>`<br>• `$dumpfile("dump.vcd");` sets file; `$dumpvars(0, tb_name);` dumps all signals.<br>• `$display` prints once; `$monitor` prints **whenever any monitored signal changes**.<br>• `$finish` terminates simulation; `$stop` pauses simulation. |
| **Lab 2: Gate Primitives** | • Syntax: `primitive_name instance_name (OUTPUT, input1, input2, ...);`<br>• **First port is ALWAYS the output!** (e.g., `nand g1(y, a, b);` $\rightarrow$ `y` is output).<br>• Built-in gates: `and`, `nand`, `or`, `nor`, `xor`, `xnor`, `buf`, `not`. (There is **no** `mux` primitive!). |
| **Lab 3: MUX, Decoder, Priority Encoder** | • **$2^n : 1$ MUX**: $n$ select lines (4:1 needs 2 select lines; $S_1S_0 = 10_2 = 2 \rightarrow D_2$).<br>• **2:4 Decoder**: 2 inputs $\rightarrow$ 4 outputs. If Enable = 0, all outputs are inactive.<br>• **Priority Encoder (4:2 or 8:3)**: Highest index has highest priority; includes a `valid` output flag indicating if any input is active. |
| **Lab 4: Dataflow Modeling** | • Keyword: `assign`<br>• **LHS of `assign` MUST be a `wire` (net), NEVER a `reg`**.<br>• Conditional operator: `assign Y = (sel) ? A : B;`<br>• Bitwise: `&` (AND), `|` (OR), `^` (XOR), `~` (NOT). |
| **Lab 5: Sequential Circuits** | • **LHS inside `always` block MUST be a `reg`**.<br>• Flip-Flops use edge trigger: `always @(posedge clk or negedge rst)`.<br>• Use **non-blocking `<=`** for sequential logic (registers/FFs) and **blocking `=`** for combinational logic.<br>• **NOR SR Latch**: $S=0, R=0 \rightarrow$ Hold; $S=1, R=0 \rightarrow Q=1$; $S=1, R=1 \rightarrow$ **Invalid/Forbidden**.<br>• **NAND SR Latch**: active-low; $S=0, R=0 \rightarrow$ Invalid. |
| **Lab 6: Full & Ripple Carry Adders** | • **Half Adder**: $\text{Sum} = A \oplus B$, $\text{Carry} = A \cdot B$.<br>• **Full Adder**: $\text{Sum} = A \oplus B \oplus C_{in}$, $C_{out} = AB + C_{in}(A \oplus B)$. (Built from 2 Half Adders + 1 OR gate).<br>• **8-bit RCA**: 8 Full Adders cascaded; delay is proportional to $N$ bits (worst case carry propagation). |
| **Lab 7: 16-bit ALU** | • Standard 2-bit opcodes: `2'b00` = ADD, `2'b01` = SUB, `2'b10` = AND, `2'b11` = OR.<br>• Status flags: Zero Flag ($Z = 1$ when result is 0), Overflow/Carry. |
| **Lab 8: 32-bit Comparators** | • Input bus syntax: `input [31:0] A, B;` (32 bits, 0 to 31).<br>• Operators: `==` (equal), `!=` (not equal), `<`, `>`, `<=`, `>=`.<br>• Outputs are 1-bit boolean (`A_gt_B`, `A_eq_B`, `A_lt_B`). Exactly one is 1 at a time. |
| **Lab 9: Register File (8 × 16-bit)** | • Depth: 8 registers $\rightarrow$ needs **3 address bits** ($2^3 = 8$).<br>• Width: 16-bit data bus (`[15:0]`).<br>• Memory declaration: `reg [15:0] reg_file [7:0];`<br>• Synchronous write on `posedge clk` when `write_en = 1`; reads are usually asynchronous. |

---

## 2. Sized Number Literals Syntax

In Verilog, constants follow the format:

$$\Large \mathbf{<\text{size}>'\!<\text{base}><\text{value}>}$$

In **`1'b1`**:
1. **The `1` before `'b`** = **Size (Bit width)** (1 bit wide).
2. **The `'b`** = **Base / Radix** (`'b` = binary, `'d` = decimal, `'h` = hex, `'o` = octal).
3. **The `1` after `'b`** = **Value** (actual logic bit).

### Quick Reference:
| Constant | Bit Width | Base | Value in Decimal |
| :--- | :---: | :---: | :---: |
| **`1'b0`** | 1 bit | Binary | `0` |
| **`1'b1`** | 1 bit | Binary | `1` |
| **`4'b1010`** | 4 bits | Binary | `10` |
| **`3'd5`** | 3 bits | Decimal | `5` (`101` binary) |
| **`8'hFF`** | 8 bits | Hexadecimal | `255` (`11111111` binary) |
| **`32'hDEADBEEF`** | 32 bits | Hexadecimal | 8 hex digits (each hex digit = 4 bits) |

---

## 3. LAB 0: Icarus Verilog Toolkit & Testbench Basics

### 1. The 3-Step Simulation Flow
1. **Compile**: `iverilog -o sim design.v tb.v`
   - `-o sim` specifies the output binary executable.
2. **Run**: `vvp sim`
   - `vvp` is the Icarus runtime engine.
3. **View**: `gtkwave dump.vcd`
   - `gtkwave` is the waveform viewer.

### 2. System Tasks
* `$dumpfile("dump.vcd");`: Selects target `.vcd` waveform dump file.
* `$dumpvars(0, tb);`: Dumps all signal variations inside module `tb` (`0` means all hierarchy levels).
* `$display("...");`: Prints once to console, adds a newline automatically.
* `$monitor("time=%0t A=%b", $time, A);`: Automatically prints **whenever any monitored signal changes value**. Only called once.
* `$finish;`: Ends simulation and terminates.
* `$stop;`: Pauses simulation and opens interactive prompt.

### 3. Testbench Port & Variable Rules
* Testbenches have **NO inputs and NO outputs** in their port list: `module tb;`.
* Driving inputs to the Unit Under Test (UUT) are declared as **`reg`**.
* Outputs observed from the UUT are declared as **`wire`**.
* Timescale: `` `timescale 1ns / 1ps `` (Unit = 1ns, Precision = 1ps).

---

## 4. LAB 2: Logic Gates and Structural Circuits

### 1. Built-in Gate Primitives
* Multi-input: `and`, `nand`, `or`, `nor`, `xor`, `xnor`
* Single-input: `not`, `buf`
* **Trap**: There is **NO** `mux`, `adder`, or `flipflop` primitive in Verilog!

### 2. Port Order Rule
$$\mathbf{\text{gate\_type}\quad \text{instance\_name}\; (\mathbf{OUTPUT},\; \text{input}_1,\; \text{input}_2,\; \dots);}$$
* The **FIRST** terminal inside the parentheses is ALWAYS the **OUTPUT**.
  ```verilog
  nand g1 (y, a, b);   // y is OUTPUT; a and b are inputs.
  ```

### 3. Truth Table Patterns $(A, B = 00, 01, 10, 11)$
| Gate | Output Pattern | Logic Rule |
| :--- | :---: | :--- |
| **AND** | `0 0 0 1` | 1 only if all inputs are 1 |
| **OR** | `0 1 1 1` | 1 if at least one input is 1 |
| **NAND** | `1 1 1 0` | 0 only if all inputs are 1 |
| **NOR** | `1 0 0 0` | 1 only if all inputs are 0 |
| **XOR** | `0 1 1 0` | 1 if inputs are **different** (odd parity) |
| **XNOR** | `1 0 0 1` | 1 if inputs are **identical** (equivalence) |
| **NOT** | `1 0` (for $A=0, 1$) | Inverter |

### 4. Wire Rule
* Intermediate connections between gates inside a module must be declared as **`wire`**.
* For an exhaustive test with $n$ inputs, you need **$2^n$ test vectors**.

---

## 5. LAB 3: MUX, Decoder, and Priority Encoder

### 1. Multiplexer (MUX)
$$\mathbf{2^n \text{ Data Inputs} \implies n \text{ Select Lines} \implies 1 \text{ Output}}$$
* **4:1 MUX**: 2 select lines ($S_1, S_0$).
  * $S_1 S_0 = 00 \rightarrow D_0$
  * $S_1 S_0 = 01 \rightarrow D_1$
  * $S_1 S_0 = 10 \rightarrow D_2$
  * $S_1 S_0 = 11 \rightarrow D_3$
* **SOP Gate Count**: 2 NOT + 4 AND + 1 OR = **7 primitive gates total**.
* **8:1 MUX**: 3 select lines ($S_2, S_1, S_0$). Can be built from **two 4:1 MUXes + one 2:1 MUX**.

### 2. Decoder ($n$-to-$2^n$)
* Takes $n$ inputs and activates **exactly one** of $2^n$ outputs (one-hot).
* 2-to-4 Decoder:
  * $00 \rightarrow Y_0 = 1 \implies 0001$
  * $01 \rightarrow Y_1 = 1 \implies 0010$
  * $10 \rightarrow Y_2 = 1 \implies 0100$
  * $11 \rightarrow Y_3 = 1 \implies 1000$

### 3. Priority Encoder (8-to-3)

#### ⚡ How to solve "Input produces code" questions in 5 seconds:
1. Label bits from right to left: $A_7, A_6, A_5, A_4, A_3, A_2, A_1, A_0$.
2. Find the **leftmost `1` (highest index)**. All lower bits are ignored.
3. Convert that winning index to 3-bit binary!
   - Example 1: `10010000` $\rightarrow$ Leftmost `1` is at $A_7 \implies 7_{10} = \mathbf{111_2}$.
   - Example 2: `01010000` $\rightarrow$ Leftmost `1` is at $A_6 \implies 6_{10} = \mathbf{110_2}$.
   - Example 3: `00001001` $\rightarrow$ Leftmost `1` is at $A_3 \implies 3_{10} = \mathbf{011_2}$.

#### ⚡ The `VALID` Flag Trap:
* $\text{VALID} = A_7 + A_6 + A_5 + A_4 + A_3 + A_2 + A_1 + A_0$
* **For `00000000`**: No inputs active $\rightarrow \mathbf{(Y_2Y_1Y_0, VALID) = (000, 0)}$.
* **For `00000001`**: Pin 0 is active $\rightarrow \mathbf{(Y_2Y_1Y_0, VALID) = (000, 1)}$.

---

## 6. LAB 4: Data-Flow Modelling

### 1. The 3 Golden Rules of `assign`
1. **LHS MUST be a `wire` (net), NEVER a `reg`**.
2. **Continuous / Reactive**: RHS is re-evaluated immediately whenever any input operand changes.
3. **Concurrent**: Statement order does not matter.

### 2. Bitwise vs Logical Operators
* **Bitwise NOT (`~`)**: Inverts each individual bit.
  - If $a = 4\text{'b}1010 \implies \sim a = \mathbf{4\text{'b}0101}$.
* **Logical NOT (`!`)**: Evaluates entire value as boolean (Zero = 0/False; Non-zero = 1/True).
  - Since $a = 4\text{'b}1010$ is non-zero (True), $!a = \mathbf{1\text{'b}0}$.

### 3. Useful Operators
* **Conditional (Ternary)**: `assign y = s ? b : a;` (2:1 MUX: $s=1 \rightarrow b, s=0 \rightarrow a$).
* **Concatenation `{}`**: `{2'b10, 2'b01} = 4'b1001`.
* **Shift Left `<<` for Decoders**: `assign Y = 4'b0001 << A;`.
* **Building a 4:1 MUX from 2:1 MUXes**: Needs **three 2:1 MUXes** (Stage 1 controlled by $S_0$, Stage 2 controlled by $S_1$).

---

## 7. LAB 5: Sequential Circuits (Latches & Flip-Flops)

### 1. Latch vs Flip-Flop
* **Latch**: **Level-sensitive** (transparent while enable is high).
* **Flip-Flop**: **Edge-triggered** (samples ONLY at `posedge` $\uparrow$ or `negedge` $\downarrow$).

### 2. NOR-Based SR Latch Truth Table
```verilog
nor n1 (Q,  R, Qb);
nor n2 (Qb, S, Q);
```
| $S$ | $R$ | Next $Q$ | State |
| :---: | :---: | :---: | :--- |
| **0** | **0** | $Q$ | **Hold / Memory** |
| **0** | **1** | 0 | **Reset** |
| **1** | **0** | 1 | **Set** |
| **1** | **1** | $Q=\overline{Q}=0$ | **Invalid / Forbidden** |
* Before any input is applied in simulation, latch outputs are **`x` (unknown)**.

### 3. D Flip-Flop Rules
* Target variable inside `always` block **must be `reg`**.
* Always use **non-blocking operator (`<=`)** inside clocked sequential blocks.
* **Synchronous Reset**: inside `always @(posedge clk)` block.
* **Asynchronous Reset**: `always @(posedge clk or posedge rst)` (reset is in sensitivity list).
* **Enabled D-FF**: Implemented using a 2:1 MUX ($D_{in} = \text{EN} \;?\; D : Q$).

### 4. Testbench Clock
```verilog
always #5 clk = ~clk; // Period = 10ns, first rising edge at t = 5ns
```

---

## 8. LAB 6: Full Adder & 8-Bit Ripple-Carry Adder (RCA)

### 1. Boolean Formulas
* **Half Adder (HA)**: $S = A \oplus B$, $C = A \cdot B$ (2 gates: 1 XOR + 1 AND).
* **Full Adder (FA)**: $S = A \oplus B \oplus C_{in}$, $C_{out} = A\cdot B + C_{in}(A \oplus B)$.
  - Built from **2 Half Adders + 1 OR gate** ($C_{out} = c_1 \mid c_2$).
  - **Total Gate Count in FA = 5 gates** (2 XOR + 2 AND + 1 OR).

### 2. Ripple-Carry Adder (RCA)
* Cascades 8 Full Adders.
* Carry ripples through all stages $\implies \mathbf{\text{Delay} \propto N}$ (linear with bit-width; slower than Carry-Lookahead).
* Carry bus: `wire [8:0] c;` where `c[0] = Cin` and `c[8] = Cout`.

### 3. Mental Math Addition Shortcut for 8-Bit MCQs
Total = $A + B + C_{in}$
* If **Total $\le 255$**: Fits in 8 bits $\rightarrow \mathbf{C_{out} = 0, \text{Sum} = \text{Total}}$.
* If **Total $\ge 256$**: Overflows 8 bits $\rightarrow \mathbf{C_{out} = 1, \text{Sum} = \text{Total} - 256}$.
  - `11111111` ($255$) + `00000001` ($1$) + $C_{in}=0 \rightarrow 256 \implies \mathbf{Sum=00000000, Cout=1}$.
  - `10101010` ($170$) + `01010101` ($85$) + $C_{in}=0 \rightarrow 255 \implies \mathbf{Sum=11111111, Cout=0}$.
  - `11110000` ($240$) + `00001111` ($15$) + $C_{in}=1 \rightarrow 256 \implies \mathbf{Sum=00000000, Cout=1}$.

---

## 9. LAB 7: 16-Bit Arithmetic Logic Unit (ALU)

### 1. Opcode Table
| `op[1:0]` | Operation | Type |
| :---: | :---: | :---: |
| **`00`** | **ADD** (`i0 + i1`) | Arithmetic |
| **`01`** | **SUB** (`i0 - i1`) | Arithmetic |
| **`10`** | **AND** (`i0 & i1`) | Logic |
| **`11`** | **OR** (`i0 | i1`) | Logic |

### 2. Adder Re-use for Subtraction
* Subtraction is $A + (\sim B) + 1$.
* Effective $B$: `b_eff = i1 ^ {16{op[0]}}`.
* $C_{in} = \text{op}[0]$.
* Temporary sum wire width: `wire [16:0] sum;` (**17 bits**: `sum[15:0]` is output, `sum[16]` is `carry`).

### 3. Carry vs Overflow
* **Carry**: Unsigned overflow (`sum[16]`).
  - In Subtraction: `carry = 1` means **NO borrow**; `carry = 0` means **borrow occurred**.
* **Overflow**: Signed overflow ($C_{15} \oplus C_{16}$).
  - Occurs when two operands of the same sign yield a result with the opposite sign.
* **Key Examples**:
  - `16'h7FFF + 16'h0001` ($+32767 + 1 = -32768$): **`o=8000, carry=0, overflow=1`**.
  - `16'hFFFF + 16'h0001` ($65535 + 1 = 65536$): **`o=0000, carry=1, overflow=0`**.
  - `16'h0005 - 16'h0007` ($5 - 7 = -2$): **`o=FFFE, carry=0`** (borrow).

---

## 10. LAB 8: 32-Bit Comparators

### 1. Relational Flags
* Inputs: `input [31:0] A, B;`
* Outputs: `eq, ne, gt, ge, lt, le` (each **1 bit**).
* Exactly one of `gt`, `eq`, `lt` is 1 at any time (mutually exclusive).
* If $A = B \implies \mathbf{eq = 1, ge = 1, le = 1}$.
* Equivalences: `ge = ~lt = gt | eq`, `le = ~gt = lt | eq`.

### 2. Signed vs Unsigned (Default is UNSIGNED)
* For $A = \mathbf{32\text{'h}FFFFFFFF}$ and $B = \mathbf{32\text{'h}00000001}$:
  - Default unsigned `A > B` $\implies \mathbf{1\text{ (TRUE)}}$ (4.29 billion > 1).
  - Signed `$signed(A) > $signed(B)` $\implies \mathbf{0\text{ (FALSE)}}$ ($-1 > 1$ is false).

### 3. `==` vs `===`
* `==` returns `x` if either operand contains `x` or `z`.
* `===` (Case equality) checks exact bit-for-bit match including `x` and `z`, returning `0` or `1`.
  - `4'b10x1 === 4'b10x1` evaluates to **`1`**.
* Gate-level equality: `assign eq = ~|(A ^ B);`.

### 4. Verification
* Two 32-bit operands have $2^{64}$ combinations $\implies$ Exhaustive testing is impossible.
* Use directed corner cases + `$random` (returns a 32-bit signed integer).

---

## 11. LAB 9: Register File (8 × 16-Bit)

### 1. Architecture
* Capacity: **8 registers × 16 bits**.
* Address width: $\log_2(8) = \mathbf{3\text{ bits}}$ (`[2:0]`).
* Ports: **2 Read Ports** (`rd_addr_a`, `rd_addr_b`) and **1 Write Port** (`wr_addr`, `d_in`, `wr`).
* Purpose: Provides small, ultra-fast storage near the ALU to avoid slow RAM access.

### 2. The $R_0$ Rule (HARD-WIRED ZERO)
* **$R_0$ is always 0**: Reading address 0 returns **`16'h0000`**.
* **Writing to address 0 is ignored**: Never changes from 0.
* **Flip-Flop Count**: $R_0$ requires 0 flip-flops.
  - Registers 1 to 7 require: $\mathbf{7 \times 16 = 112\text{ flip-flops}}$!

### 3. Structural Implementation
* Built from `dfrl` (D-FF with reset and load enable).
* Common pins: Global `clk`, global `reset`, and `d_in`.
* Write logic: Decoded `wr_addr` gated with `wr`: `load[i] = wr & (wr_addr == i)`.
* Read logic: Two **8:1 Multiplexers (16-bit wide)**.
* Array declaration: `reg [15:0] regs [0:7];` (word width first, depth second).

### 4. Read/Write Timing
* Write occurs on `posedge clk` when `wr = 1`.
* Simultaneous read and write on the same cycle: due to non-blocking assignment (`<=`), the read returns the **OLD value**; new data appears on the subsequent clock cycle.

