# Verilog HDL Fundamentals for Digital Design and Functional Verification
### Study Notes — Course by Ovidiu Plugariu (Udemy)

> Instructor background: PhD in Electronics & Telecommunications, 10+ years as a Design & Verification engineer, 19,000+ hours of hands-on Verilog across microcontroller/ASIC/FPGA design and verification for consumer electronics, automotive, aerospace, and academia.

---

## Table of Contents
1. [Introduction & Design Flow](#1-introduction--design-flow)
2. [Abstraction Levels](#2-abstraction-levels)
3. [Simulation & EDA Tooling](#3-simulation--eda-tooling)
4. [Verilog Data Types & Literals](#4-verilog-data-types--literals)
5. [Operators](#5-operators)
6. [Modules & Hierarchy](#6-modules--hierarchy)
7. [Testbench Fundamentals](#7-testbench-fundamentals)
8. [Design Styles: Structural, Dataflow, Behavioral](#8-design-styles-structural-dataflow-behavioral)
9. [Combinational Design](#9-combinational-design)
10. [Sequential Design](#10-sequential-design)
11. [Shift Registers](#11-shift-registers)
12. [Counters & Frequency Dividers](#12-counters--frequency-dividers)
13. [Functions & Tasks](#13-functions--tasks)
14. [Self-Checking Testbenches & Golden Models](#14-self-checking-testbenches--golden-models)
15. [Memory Design: SRAM / DRAM / ROM](#15-memory-design-sram--dram--rom)
16. [Finite State Machines (FSM)](#16-finite-state-machines-fsm)
17. [FIFO Design](#17-fifo-design)
18. [Capstone: Data-Transfer Bridge FSM](#18-capstone-data-transfer-bridge-fsm)
19. [Data Encryption Module (Stream Cipher)](#19-data-encryption-module-stream-cipher)
20. [Keyword / System Task / Operator Glossary](#20-keyword--system-task--operator-glossary)
21. [Acronym Glossary](#21-acronym-glossary)
22. [Deep Dive: Simulation Event Scheduling](#22-deep-dive-simulation-event-scheduling)
23. [Cheat Sheet: Rules of Thumb](#23-cheat-sheet-rules-of-thumb)

---

## 1. Introduction & Design Flow

Verilog HDL (Hardware Description Language) is used to **describe the structure and behavior of digital electronic circuits**, enabling both **simulation** and **synthesis**.

- Invented in 1985; first standardized in **1995** as **IEEE 1364-1995**.
- Later revisions: **Verilog-2001** (added generate blocks, always @*, signed arithmetic improvements) and **Verilog-2005**.
- Verilog is the forerunner of **SystemVerilog** (IEEE 1800), which extended it heavily for verification (classes, interfaces, constrained-random, assertions).

> [!NOTE]
> The course is taught using **Intel Quartus Prime Lite** (synthesis/FPGA flow) + **ModelSim Intel FPGA Edition** (simulation). Industry-grade equivalents mentioned: ModelSim/Questa (Siemens/Mentor), Incisive/Xcelium (Cadence), VCS (Synopsys), Vivado (Xilinx/AMD), and the open-source Icarus Verilog.

**Key properties of Verilog:**
- Syntax is C-like (operator precedence, control-flow keywords).
- Designs are partitioned into a **hierarchy of modules** that communicate via `input`/`output`/`inout` ports.
- Describes **concurrent (parallel) processes** — a fundamental difference from sequential software languages.
- Can model behavior at **multiple abstraction levels**.

---

## 2. Abstraction Levels

Abstraction levels let engineers specify *what* a system does without drowning in *how* it's implemented — each level carries only the detail relevant to that stage.

```
System Level        (highest)   e.g. "LAN speed shall be 10 Gbps"
   |
Algorithm Level                 e.g. "CPU averages sensor samples, sends to server"
   |
Register Transfer Level (RTL)   e.g. "Encryption output stored in 128b x 64 sync FIFO"
   |
Gate Level                      e.g. "Synthesize using Fab 32/65 gate library"
   |
Circuit Level                   e.g. transistor-level schematic of an AND gate
   |
Material Level       (lowest)   e.g. "Cells implemented in 32nm process"
```

> [!NOTE]
> Verilog's practical sweet spot is the **Algorithm, RTL, and Gate** levels (the "blue rectangle" in the course diagram). Its modeling capability at System and Circuit level is limited — those are handled by other tools/languages (SystemC, SPICE, etc.).

---

## 3. Simulation & EDA Tooling

**EDA (Electronic Design Automation)** = software used to design ICs and PCBs, spanning specification, design, implementation, verification, and test.

- **CAD** (Computer-Aided Design) → creating schematics.
- **CAE** (Computer-Aided Engineering) → analyzing schematics/behavior.

**EDA tool categories covered:**
- Chip design & verification tools
- PCB / multi-chip module tools
- Semiconductor IP tools
- Services tools

### Basic ModelSim Project Flow (used throughout the course)
1. Create a project folder (e.g. `action_time_1`) containing your `.v` design file(s).
2. Create a `sim` subfolder for the ModelSim project.
3. **File → New → Project** → browse to `sim` folder → OK.
4. **Add Existing File** → browse up a level → select your `.v` file(s) → OK → Close.
5. Right-click → **Compile → Compile All** (green check = success).
6. **Simulate → Start Simulation** → expand `work` library → select top module → OK.
7. Drag signals into the **Wave** window (Ctrl+A to select all, drag & drop). Toggle leaf names for readability.
8. Click **Run All**.
9. To re-run after edits: recompile → **Restart** → OK → **Run All**.

> [!TIP]
> Select signals in the Wave pane, right-click → **Radix** to switch between binary/hex/decimal/unsigned display — very useful for verifying bus values at a glance.

---

## 4. Verilog Data Types & Literals

Verilog has **three major categories of value objects**:

| Category | Purpose | Examples |
|---|---|---|
| **Nets** | Physical connections between structures (combinational + sequential logic wiring) | `wire`, `tri`, `supply0`, `supply1`, `tri0`, `tri1`, `wand`, `triand`, `wor`, `trior`, `trireg` |
| **Variables** | Abstract storage elements | `reg` (1-bit+), `integer` (32-bit signed), `real` (64-bit floating), `time` (32-bit), `realtime` (64-bit) |
| **Parameters** | Compile-time constants (like C `#define`) | `parameter`, `localparam`, `specparam` |

> [!NOTE]
> A `reg` is **not** necessarily a hardware register/flip-flop — it's just a variable that can hold a value between procedural assignments. Whether it becomes a flip-flop, latch, or pure combinational logic depends entirely on **how it's used** (inside `always @(posedge clk)` → flip-flop; inside `always @*` fully-specified → combinational logic; inside `always @*` partially-specified → latch).

### Data Type Synthesis Rule
- `wire`/`reg` are the practical, synthesizable HDL types.
- A **net** is driven by a driver's value (usually a `reg`); if certain conditions are met, `reg`/`wire` based code converts into gate-level circuits via **synthesis**.
- Rule: variables (`reg`, `integer`, `time`) can **only be driven inside procedures** (`initial`/`always`); nets are driven **everywhere else** (outside procedures, e.g. `assign` statements or gate instantiations).

### Literal Value Syntax

```
<size>'<base><value>
```
- **size** = total number of bits.
- **base** = `b` (binary), `o` (octal), `d` (decimal), `h` (hexadecimal). Prefix with `s` for signed (e.g. `sb`, `sh`).
- **value** = digits in the chosen base; unsigned by default; underscore `_` may separate bit groups for readability; `x`/`z` allowed in binary/octal/hex.

```verilog
// Example literal declarations
reg [7:0] myvar;
myvar = 8'd137;        // decimal 137
myvar = 8'h89;          // same value in hex
myvar = 8'b1000_1001;   // same value in binary, underscore for grouping
myvar = 8'o211;         // same value in octal
myvar = 8'hz1;          // upper nibble = high-impedance (Z), lower nibble = 001
```

> [!WARNING]
> Assigning a value wider than the destination truncates the **most-significant bits**; the destination keeps only its **least-significant bits** from the source value.

### Vectors (Multi-bit Signals)

- A vector is a `wire`/`reg` with a declared range, e.g. `reg [7:0] data;` (MSB on the left, LSB on the right — though descending or ascending ranges are both legal, e.g. `[3:0]` or `[-2:1]`).
- **Bit-select**: `vector[i]` picks a single bit.
- **Bit-slicing**: `vector[msb:lsb]` picks a sub-range; bounds must be constant.
- **Concatenation**: `{a, b}` joins bit fields into a new vector — usable on both LHS and RHS.
- **Replication**: `{N{value}}` repeats `value` N times (N must be constant).

```verilog
// Bit-slicing and concatenation example
wire [3:0] a, b;
wire [1:0] c;
assign c[1] = a[3];   // continuous assignment / bit-slicing
assign c[0] = a[2];

// Concatenation & swap idiom
d = {a[3:0], b[6:3]};      // concatenate two nibbles into an 8-bit value
{a, b} = {b, a};            // swap two vectors of equal width using concatenation

// Replication examples
a = {4{2'b10}};      // replicate 2-bit value '10' four times -> 8 bits
b = {8{4'b1x0z}};    // replication also accepts x/z bits
```

---

## 5. Operators

Verilog operator classes: **bitwise, reduction, logical, arithmetic, shift, relational, equality, conditional, concatenation, replication.**

### Bitwise Operators (operate per-bit on scalars or vectors)
| Operator | Meaning |
|---|---|
| `&` | AND |
| `\|` | OR |
| `^` | XOR |
| `~&` | NAND |
| `~\|` | NOR |
| `~^` / `^~` | XNOR |
| `~` | NOT (unary) |

### Logical Operators (return a single 1-bit true/false/unknown)
| Operator | Meaning |
|---|---|
| `!` | logical NOT |
| `&&` | logical AND |
| `\|\|` | logical OR |

> [!TIP]
> **Best practice:** use logical operators (`&&`, `\|\|`, `!`) only inside conditional statements (`if`, `while`) — never assign their result directly to a wide bus, since that would incorrectly collapse a multi-bit comparison down to 1 bit driving all bits of a wider destination.

### Arithmetic Operators
`+`, `-`, `*`, `/`, `%` (modulo), `**` (exponentiation, introduced Verilog-2001).

### Shift Operators
| Operator | Meaning | Introduced |
|---|---|---|
| `<<` | logical left shift | Verilog-1995 |
| `>>` | logical right shift | Verilog-1995 |
| `<<<` | arithmetic left shift (identical to `<<`) | Verilog-2001 |
| `>>>` | arithmetic right shift (preserves sign/MSB for two's-complement numbers) | Verilog-2001 |

> [!NOTE]
> Left shift by 1 = multiply by 2; right shift by 1 = divide by 2 (for logical/unsigned shifts). Arithmetic right-shift preserves the sign bit during the shift — critical for signed numbers.

### Relational Operators
`<`, `>`, `<=`, `>=` — return unknown (`x`) if an operand contains `x`/`z`.

### Equality Operators
| Operator | Name | Behavior with x/z | Typical usage |
|---|---|---|---|
| `==` | logical equality | returns `x` if either operand has `x`/`z` | design (RTL) |
| `!=` | logical inequality | same as above | design (RTL) |
| `===` | case equality | compares `x`/`z` literally; always returns 0 or 1 | testbench |
| `!==` | case inequality | same as above | testbench |

> [!WARNING]
> Always use **case equality (`===`/`!==`)** in self-checking testbenches when comparing DUT outputs against expected values, since a DUT bug that produces `x`/`z` would otherwise silently evaluate to `x` (neither pass nor clean fail) with `==`.

### Concatenation & Replication
```verilog
assign a = {b, c};        // concatenation: {6-bit A} = {4-bit B, 2-bit C}
assign a = {1'b1, a[6:0] << 1}; // shift left and insert new LSB
assign a = {4{2'b10}};    // replication
```

### Operator Precedence (highest → lowest, abbreviated)
```
1. Unary (!, ~, +, -, & (reduction), | (reduction), ^ (reduction) ...)
2. **  (exponentiation)
3. * / %
4. + -
5. << >> <<< >>>
6. < <= > >=
7. == != === !==
8. & (bitwise)
9. ^ ^~
10. |  (bitwise)
11. &&
12. ||
13. ?: (conditional)
14. {} (concatenation) — lowest
```
> [!TIP]
> Always use **parentheses** for clarity even when precedence rules would work — this is called out repeatedly in the course as an industry best practice and prevents subtle bugs (e.g. `2 << 0` without parentheses being ambiguous to read even though it evaluates correctly).

### Reduction Operators
Applied to a single vector, collapsing all its bits down to 1 bit: `&`, `~&`, `|`, `~|`, `^`, `~^`.

---

## 6. Modules & Hierarchy

A **module** is the fundamental building block of Verilog — analogous to a class/function unit that implements specific functionality (an adder, an ALU, a testbench, etc.).

**Two basic rules taught in the course:**
1. Each module should be written in a file with the **same name** as the module.
2. All Verilog code is encapsulated within `module ... endmodule`.

```verilog
// Synthesizable — Structural/Dataflow style
// 8-bit adder module
module adder_8bit(
    input  [7:0] a,
    input  [7:0] b,
    output [8:0] sum   // 9 bits to capture carry-out
);
    assign sum = a + b;
endmodule
```

### Port Connectivity Rules
- `input` ports must be **nets** (`wire` type) as seen from inside the module.
- `output` ports can be **nets or registers** (`wire` or `reg`), depending on whether they're driven by continuous assignment or a procedural block.
- Each output port is a **source**; each input port is a **destination**.

### Hierarchy (Russian-Doll Analogy)
```
Top-Level Module
 └── Sub-module A
      └── Sub-sub-module A1
 └── Sub-module B
```
- The **top-level module** instantiates all sub-modules.
- Complex functionality is split into smaller, testable/maintainable modules — this is **top-level integration**, a real design stage in every modern chip.
- A **testbench** is a special module with **no ports** — it instantiates the **DUT (Design Under Test)**, generates stimulus for inputs, and monitors outputs.

```
        ┌─────────────────────────────┐
        │         TESTBENCH           │
        │  (no ports, generates       │
        │   stimulus, monitors DUT)   │
        │                              │
        │   ┌──────────────────────┐  │
        │   │   DUT (instantiated)  │  │
        │   │  in -> [ logic ] -> out│  │
        │   └──────────────────────┘  │
        └─────────────────────────────┘
```

---

## 7. Testbench Fundamentals

### Standard Testbench Recipe (repeated throughout the course)
1. Declare testbench variables: `reg` for DUT **inputs**, `wire` for DUT **outputs**.
2. Instantiate the DUT, connecting ports **by name** (`.port_name(tb_variable)`).
3. Use `$monitor` or `$display` to observe signal changes.
4. Generate stimulus in an `initial` block.

```verilog
// Non-synthesizable — Testbench for 8-bit adder
module tb_adder_8bit;
    reg  [7:0] a, b;
    wire [8:0] sum;

    // Instantiate DUT — dot-name connection
    adder_8bit dut (
        .a(a),
        .b(b),
        .sum(sum)
    );

    initial begin
        $monitor("time=%0t a=%d b=%d sum=%d", $time, a, b, sum);
        a = 8'd10; b = 8'd20;
        #10 a = 8'd100; b = 8'd50;
        #10 $finish;
    end
endmodule
```

> [!NOTE]
> **Input variables** in a testbench must be `reg` type (they are driven procedurally); **output variables** must be `wire` type (they are driven by the DUT, i.e. from outside the testbench's own procedural code).

### Timescale
```verilog
`timescale 1ns/1ps   // time unit = 1ns, time precision = 1ps
```
- Simulation always starts at time 0 and runs until a stop command (`$finish` or `$stop`) or the testbench's own control logic ends it.
- Example: an 8-period clock with a 3ns period → total waveform duration = 24ns.

---

## 8. Design Styles: Structural, Dataflow, Behavioral

| Style | Keyword usage | Data type on LHS | Best for |
|---|---|---|---|
| **Structural** | module instantiation, built-in primitives (`and`, `or`, `xor`, ...) | `wire` | Top-level integration, gate-level netlists |
| **Dataflow** | `assign` (continuous assignment) | `wire` (must be a net) | Simple combinational boolean/arithmetic expressions |
| **Behavioral** | `always`/`initial` procedural blocks | `reg` | Complex combinational or sequential logic, higher abstraction |

### Structural Style Example (Half Adder using built-in primitives)
```verilog
// Synthesizable — Structural style, built-in primitives
module half_adder_structural(
    input  a, b,
    output sum, carry
);
    xor (sum, a, b);     // built-in primitive: output listed first
    and (carry, a, b);
endmodule
```

### Dataflow Style Example (same Half Adder)
```verilog
// Synthesizable — Dataflow style
module half_adder_dataflow(
    input  a, b,
    output sum, carry
);
    assign sum   = a ^ b;
    assign carry = a & b;
endmodule
```

### Behavioral Style Example (same Half Adder)
```verilog
// Synthesizable — Behavioral style
module half_adder_behavioral(
    input  a, b,
    output reg sum, carry   // must be `reg` — driven inside always block
);
    always @(*) begin
        sum   = a ^ b;
        carry = a & b;
    end
endmodule
```

> [!NOTE]
> All three styles are functionally identical (same testbench + same stimulus yields identical results) — they only differ in **how** the behavior is described. Black-box testing (testing a DUT purely on I/O behavior, ignoring internal implementation) works identically regardless of style.

### `initial` vs `always` Procedural Blocks

| Block | Starts | Ends | Typical use |
|---|---|---|---|
| `initial` | at simulation time 0 | after all statements execute once (sequentially) | testbench stimulus generation |
| `always` | at simulation time 0 | never (loops forever, re-triggered by its sensitivity list) | synthesizable combinational/sequential logic, or a `forever` in a testbench |

- Multiple `initial`/`always` blocks execute **concurrently** (in parallel) — this models real hardware where all logic operates simultaneously.
- **Sensitivity list** (a.k.a. **event control list**) — the set of signals that trigger re-execution of an `always` block: `@(a or b)`, or with the wildcard `@(*)` (a.k.a. `always @*`).

> [!WARNING]
> **Always use the wildcard operator `@(*)` for combinational logic.** If you hand-list an explicit sensitivity list and forget a signal, the resulting hardware model will fail to update when that signal changes in real hardware, but simulation with the incomplete list can silently mask the bug — a classic **simulation/synthesis mismatch**.

### Full Adder from Two Half Adders (Structural)
```verilog
// Synthesizable — Structural, hierarchical (3-level hierarchy example)
module full_adder_structural(
    input  a, b, cin,
    output sum, cout
);
    wire s1, c1, c2;

    half_adder_structural ha1(.a(a),  .b(b),  .sum(s1), .carry(c1));
    half_adder_structural ha2(.a(s1), .b(cin),.sum(sum),.carry(c2));
    or (cout, c1, c2);
endmodule
```

```
Block-level diagram of full_adder_structural:

  a ──┐
      ├──[HA1]── s1 ──┐
  b ──┘        └c1     ├──[HA2]── sum
                        │      └c2
 cin ────────────────────┘
                         c1, c2 ──[OR]── cout
```

---

## 9. Combinational Design

**Combinational logic**: output is purely a function of *current* inputs; no memory; time-independent; consists only of logic gates; propagation delay is the only "speed" limiter. Examples: encoders, decoders, MUX/DEMUX, adders/subtractors, comparators, ALUs.

### Continuous Assignments (`assign`)
- Used **only** for combinational logic.
- Must be written **outside** procedural blocks.
- LHS **must be a net** (`wire`).
- Multiple `assign` statements execute **in parallel**, in any order — never assign the same net from more than one continuous assignment.

```verilog
// Synthesizable — Dataflow, parameterized adder tree
module adder_tree(
    input  [3:0] a, b,
    input  [7:0] c, d,
    input  [4:0] e,
    input  [8:0] f,
    output [4:0] sum1,   // a + b
    output [8:0] sum2,   // c + d
    output [9:0] sum3    // e + f
);
    assign sum1 = a + b;
    assign sum2 = c + d;
    assign sum3 = e + f;
endmodule
```

### Procedural (Behavioral) Combinational Logic
```verilog
// Synthesizable — Behavioral, same adder tree using always blocks
module adder_tree_behavioral(
    input  [3:0] a, b,
    output reg [4:0] sum1
);
    always @(a or b)         // explicit sensitivity list (works, but not best practice)
        sum1 = a + b;

    // Preferred: always @(*) sum1 = a + b;
endmodule
```

> [!TIP]
> **Blocking (`=`) vs Non-blocking (`<=`) assignment rule:**
> - Use **blocking (`=`)** for **combinational** logic (`always @(*)`).
> - Use **non-blocking (`<=`)** for **sequential** logic (`always @(posedge clk)`).
> - Mixing these up is one of the most common sources of subtle, hard-to-debug simulation/synthesis mismatches.

```verilog
always @(*) begin              // COMBINATIONAL
    variable = expression;     // blocking
end

always @(posedge clk) begin    // SEQUENTIAL
    variable <= expression;    // non-blocking
end
```

### Parameterized (Generic) Modules
`parameter` allows a module to be reused for different bit-widths, resolved at **compile/elaboration time** (not runtime).

```verilog
// Synthesizable — Parameterized N-bit adder
module adder_n #(parameter N = 8)(
    input  [N-1:0] a, b,
    output [N:0]   sum
);
    assign sum = a + b;
endmodule

// Instantiation with parameter override
adder_n #(.N(10)) adder1 (.a(a), .b(b), .sum(sum));  // creates a 10-bit adder
```

### N-bit Comparator (Behavioral, Parameterized)
```verilog
// Synthesizable — Behavioral
module comparator_n #(parameter N = 4)(
    input  [N-1:0] a, b,
    output reg smaller, equal, greater
);
    always @(*) begin
        smaller = (a < b);
        equal   = (a == b);
        greater = (a > b);
    end
endmodule
```

### Decoder with Enable (Parameterized, Case Statement)
```verilog
// Synthesizable — Behavioral, using a case statement + default (latch prevention)
module decoder_n #(parameter N = 3)(
    input  [N-1:0] a,
    input          enable,
    output reg [(1<<N)-1:0] y
);
    always @(*) begin
        if (!enable)
            y = 0;
        else
            y = (1 << a);
    end
endmodule
```

### Building Larger Decoders From Smaller Ones (4-to-16 from two 3-to-8)
```
              ┌──────────────┐
   a[3]───┬──►│ enable(inv)  │
           │  │  DEC_3to8 #1 │──► y[7:0]
   a[2:0]──┼─►│  a           │
           │  └──────────────┘
           │
           └──►│ enable       │
    a[3]───┬──►│  DEC_3to8 #2 │──► y[15:8]
   a[2:0]──┴──►│  a           │
               └──────────────┘
```
- `a[3]` inverted enables decoder #1 for inputs 0–7; `a[3]` directly enables decoder #2 for inputs 8–15.

### Priority Encoder (if-else-if vs case)
```verilog
// Synthesizable — Behavioral, priority encoder using if-else-if (D3 highest priority)
module prio_enc_4to2_ifelse(
    input  [3:0] d,
    output reg [1:0] q,
    output reg v
);
    always @(*) begin
        if      (d[3]) q = 2'd3;
        else if (d[2]) q = 2'd2;
        else if (d[1]) q = 2'd1;
        else if (d[0]) q = 2'd0;
        else            q = 2'd0;

        v = (d != 4'b0000);
    end
endmodule
```
```verilog
// Synthesizable — same priority encoder using casez/case-inside-priority style
module prio_enc_4to2_case(
    input  [3:0] d,
    output reg [1:0] q,
    output reg v
);
    always @(*) begin
        v = 1'b1;
        case (1'b1)     // "case(1'b1)" idiom = evaluate first true condition
            d[3]: q = 2'd3;
            d[2]: q = 2'd2;
            d[1]: q = 2'd1;
            d[0]: q = 2'd0;
            default: begin q = 2'd0; v = 1'b0; end
        endcase
    end
endmodule
```

> [!WARNING]
> Priority-encoding conditions **must be ordered from highest to lowest priority** — reversing the order silently changes functional behavior without any syntax error.

### Bus Multiplexer / Demultiplexer (Case-based)
```verilog
// Synthesizable — Parameterized 4-to-1 bus mux
module mux4 #(parameter N = 8)(
    input  [N-1:0] a, b, c, d,
    input  [1:0]   sel,
    output reg [N-1:0] y
);
    always @(*) begin
        case (sel)
            2'd0: y = a;
            2'd1: y = b;
            2'd2: y = c;
            2'd3: y = d;
            default: y = {N{1'b0}};
        endcase
    end
endmodule
```

```verilog
// Synthesizable — Parameterized 1-to-4 bus demux
module demux4 #(parameter N = 8)(
    input  [N-1:0] y,
    input  [1:0]   sel,
    output reg [N-1:0] a, b, c, d
);
    always @(*) begin
        a = 0; b = 0; c = 0; d = 0;    // default values (avoid unintended latch!)
        case (sel)
            2'd0: a = y;
            2'd1: b = y;
            2'd2: c = y;
            2'd3: d = y;
            default: ;
        endcase
    end
endmodule
```

> [!TIP]
> Setting **default values at the top of a combinational always block** (before the `case`) is a widely-used pattern to guarantee every output is assigned on every path, preventing accidental latch inference.

### Tri-State Buffer / Bus (Structural Primitive)
```verilog
// Synthesizable — single tri-state buffer using built-in primitive
module tristate_buf_1bit(
    input  d_in, sel,
    output d_out
);
    bufif1 (d_out, d_in, sel);   // drives d_in to d_out only when sel=1, else Z
endmodule
```
- Truth table: `sel=0` → `d_out = Z` (high-impedance, electrically disconnected); `sel=1` → `d_out = d_in`.
- Used for bidirectional chip I/O pads and shared PCB/board buses.

### 7-Segment Display Decoder
- Converts a 4-bit BCD/hex nibble into 7 (or 8 with decimal point) segment-driving outputs (`a`–`g`, optionally `dp`).
- **Common cathode**: segment ON with logic `1`. **Common anode**: segment ON with logic `0` (bit-invert the common-cathode code).
- Reference IC named in the course: **CD4511B** (BCD-to-7-segment decoder, Texas Instruments).

### ALU (Arithmetic Logic Unit)
- First single-chip ALU referenced: **Intel/TI 74181** — historically significant for shrinking multi-cabinet computers down to a single PCB.
- Course ALU: 8-bit operands, 4-bit opcode, 8-bit result + flags (**carry-out, borrow, zero, parity, invalid-operation**).

```verilog
// Synthesizable — Behavioral 8-bit ALU (illustrative, condensed)
module alu #(parameter WIDTH = 8)(
    input  [WIDTH-1:0] a, b,
    input  [3:0] opcode,
    output reg [WIDTH-1:0] y,
    output reg carry_out, borrow, zero, parity, invalid_op
);
    localparam OP_ADD  = 4'd0, OP_ADDC = 4'd1, OP_SUB  = 4'd2,
               OP_INC  = 4'd3, OP_DEC  = 4'd4, OP_AND  = 4'd5,
               OP_XOR  = 4'd6, OP_NOT  = 4'd7, OP_ROL  = 4'd8, OP_ROR = 4'd9;

    always @(*) begin
        {carry_out, y} = {1'b0, {WIDTH{1'b0}}};
        borrow = 0; invalid_op = 0;
        case (opcode)
            OP_ADD:  {carry_out, y} = a + b;                 // NOTE: carry_out must come
                                                               // from the extended-width add
            OP_ADDC: {carry_out, y} = a + b + carry_out;
            OP_SUB:  {borrow, y}    = a - b;
            OP_INC:  y = a + 1'b1;     // 1'b1, not 1, to force N-bit adder (not 32-bit int add)
            OP_DEC:  y = a - 1'b1;
            OP_AND:  y = a & b;
            OP_XOR:  y = a ^ b;
            OP_NOT:  y = ~a;
            OP_ROL:  y = {a[WIDTH-2:0], a[WIDTH-1]};
            OP_ROR:  y = {a[0], a[WIDTH-1:1]};
            default: invalid_op = 1'b1;
        endcase
        zero   = (y == 0);
        parity = ^y;   // XOR-reduction: 1 if odd number of 1-bits
    end
endmodule
```

> [!WARNING]
> **Known bug pattern taught in the course:** if `OP_ADD` is coded as `y = a + b;` (assigning directly into the N-bit result) instead of `{carry_out, y} = a + b;`, the **carry-out overflow is silently dropped** — this was an intentionally-inserted "Easter egg" bug found later via a randomized, self-checking testbench with a golden model (see [Section 14](#14-self-checking-testbenches--golden-models)).

> [!TIP]
> Always write increment/decrement as `a + 1'b1` / `a - 1'b1` (single-bit literal `1`), **not** bare `a + 1`. Some synthesis tools (e.g. Quartus for certain targets) may interpret an un-sized `1` as a 32-bit integer operation, forcing a much larger adder/subtractor than needed — an area-optimization pitfall.

---

## 10. Sequential Design

**Sequential logic**: output depends on *both* current inputs *and* past history (memory); operation is governed by a **clock**.

### Clock Signal Parameters
| Term | Definition |
|---|---|
| **Period** | Time between two consecutive positive edges (seconds) |
| **Frequency** | `1 / period` (Hertz) |
| **Duty cycle** | `pulse_high_time / period` (%) |
| **Phase** | Relationship between two clocks; e.g. 90° = quarter-period offset, 180° = half-period offset |

### Clock Generation Idioms
```verilog
// Non-synthesizable — Testbench clock generation, two equivalent methods
`timescale 1us/1ns
reg clock1, clock2;
parameter PERIOD1 = 1;   // 1 MHz
parameter PERIOD2 = 0.5; // 2 MHz

// Method 1: initial + forever
initial begin
    clock1 = 0;
    forever #(PERIOD1/2) clock1 = ~clock1;
end

// Method 2: always block (equivalent, more compact)
initial clock2 = 0;          // MUST initialize, else clock2 stays X forever
always #(PERIOD2/2) clock2 = ~clock2;
```

> [!WARNING]
> When using the `always #(half_period) clk = ~clk;` clock-generation idiom, you **must** initialize the clock variable in a separate `initial` block first — otherwise it stays at `x` for the entire simulation, since `~x = x`.

### Edge-Triggered vs Level-Triggered Logic
| Type | Verilog construct | Element |
|---|---|---|
| Positive edge-triggered | `always @(posedge clk)` | Flip-flop |
| Negative edge-triggered | `always @(negedge clk)` | Flip-flop |
| Active-high level-triggered | `always @(enable or d)` w/ `if(enable)` | Latch |
| Active-low level-triggered | same, `if(!enable)` | Latch |

### D-Latch (Transparent Latch)
```verilog
// Synthesizable — Behavioral level-sensitive latch
module d_latch(
    input  d, enable,
    output reg q,
    output q_n
);
    always @(enable or d)
        if (enable) q <= d;   // non-blocking, per course convention for latching code

    assign q_n = ~q;
endmodule
```
- **Truth table:** `enable=0` → hold current value (latched); `enable=1` → transparent, `q` follows `d`.

### D-Latch with Active-Low Async Reset
```verilog
module d_latch_reset(
    input  d, enable, reset_n,
    output reg q,
    output q_n
);
    always @(reset_n or enable or d)
        if (!reset_n)    q <= 1'b0;
        else if (enable) q <= d;
    assign q_n = ~q;
endmodule
```

### D Flip-Flop — Synchronous Reset, Positive Edge
```verilog
// Synthesizable — Behavioral, synchronous active-low reset
module dff_sync_reset_n(
    input  clk, reset_n, d,
    output reg q,
    output q_n
);
    always @(posedge clk)          // reset_n NOT in sensitivity list -> synchronous
        if (!reset_n) q <= 1'b0;
        else          q <= d;

    assign q_n = ~q;
endmodule
```

### D Flip-Flop — Asynchronous Reset, Positive Edge
```verilog
// Synthesizable — Behavioral, asynchronous active-low reset
module dff_async_reset_n(
    input  clk, reset_n, d,
    output reg q,
    output q_n
);
    always @(posedge clk or negedge reset_n)   // reset_n IS in sensitivity list -> async
        if (!reset_n) q <= 1'b0;
        else          q <= d;

    assign q_n = ~q;
endmodule
```

> [!NOTE]
> **Synchronous vs asynchronous reset:** if the reset signal appears in the sensitivity list (`or negedge reset_n`), the flip-flop reacts to reset **immediately**, without waiting for a clock edge (asynchronous). If it's omitted from the sensitivity list, reset only takes effect **on the next active clock edge** (synchronous) — a subtly different hardware behavior with real implications for safety-critical designs (async reset can react even if the clock has stopped).

### Latch vs Flip-Flop — Trade-off Summary

| Property | Latch | Flip-Flop |
|---|---|---|
| Trigger | Level (enable) | Edge (posedge/negedge) |
| Built from | Logic gates | Two latches + clock |
| Speed | Faster | Slower |
| Power | Lower | Higher |
| Silicon area | ~half of a flip-flop | Larger |
| Noise sensitivity | Enable-pin noise can corrupt data | Edge-triggered → more robust between clock edges |
| Typical use | Rarely used directly in synchronous ASIC design (transparency causes timing hazards) | Standard building block of registers in virtually all digital systems |

---

## 11. Shift Registers

A **shift register** transfers/stores data using a chain of flip-flops, moving data left/right per clock cycle.

### Classification by Data Movement
| Type | Meaning |
|---|---|
| **PIPO** | Parallel-In, Parallel-Out — all bits load/read simultaneously |
| **SISO** | Serial-In, Serial-Out — one bit per clock, from one side to the other |
| **PISO** | Parallel-In, Serial-Out — loaded in parallel, shifted out serially |
| **SIPO** | Serial-In, Parallel-Out — loaded serially, read out in parallel |

### PIPO Register (Pipeline Register)
```verilog
// Synthesizable — Behavioral PIPO register with async active-low reset
module shift_reg_pipo #(parameter N = 4)(
    input  clk, reset_n,
    input  [N-1:0] d,
    output reg [N-1:0] q
);
    always @(posedge clk or negedge reset_n)
        if (!reset_n) q <= {N{1'b0}};
        else          q <= d;
endmodule
```
> [!NOTE]
> This is called a **pipeline register** in industry — used to break up large combinational paths, shortening propagation delay per pipeline stage and raising the maximum achievable clock frequency (higher throughput at the cost of added latency).

### SISO Register
```verilog
// Synthesizable — Behavioral 4-bit SISO shift register
module shift_reg_siso(
    input  clk, reset_n, sd_in,
    output sd_out
);
    reg [3:0] shreg;
    always @(posedge clk or negedge reset_n)
        if (!reset_n) shreg <= 4'b0;
        else          shreg <= {shreg[2:0], sd_in};   // shift left, insert new bit at LSB

    assign sd_out = shreg[3];   // MSB is the serial output
endmodule
```

### PISO Register (with Preload)
```verilog
// Synthesizable — Behavioral PISO with mux-per-bit conceptual model
module shift_reg_piso #(parameter N = 4)(
    input  clk, reset_n, preload, sdi,
    input  [N-1:0] d,
    output sdo
);
    reg [N-1:0] shreg;
    wire [N-1:0] data_src = preload ? d : {shreg[N-2:0], sdi};

    always @(posedge clk or negedge reset_n)
        if (!reset_n) shreg <= {N{1'b0}};
        else          shreg <= data_src;

    assign sdo = shreg[N-1];
endmodule
```
> [!NOTE]
> The extra `sdi` (serial-data-in) port lets multiple PISO modules be daisy-chained to build wider shift registers (connect one module's `sdo` to the next module's `sdi`).

### Bidirectional Shift/Load Register
```verilog
// Synthesizable — Behavioral, load / shift-left / shift-right register
module shift_left_right_load #(parameter N = 8)(
    input  clk, reset_n, load_enable, shift_left_right,
    input  [N-1:0] i,
    output reg [N-1:0] q
);
    always @(posedge clk or negedge reset_n) begin
        if (!reset_n)
            q <= {N{1'b0}};
        else if (!load_enable)
            q <= i;
        else begin
            if (!shift_left_right) q <= {q[N-2:0], 1'b0};   // shift left
            else                   q <= {1'b0, q[N-1:1]};   // shift right
        end
    end
endmodule
```

### LFSR — Linear Feedback Shift Register (Pseudo-Random Number Generator)
- Based on shifting + XOR feedback taps derived from a **primitive polynomial**.
- The starting value is called the **seed**; the seed **must never be all-zeros or all-ones** (those are degenerate lock-up states for a standard LFSR).
- A **maximal-length N-bit LFSR** produces `2^N - 1` distinct patterns before repeating.
- Applications: cryptography (stream ciphers), built-in self-test (BIST), scrambling/whitening.

```verilog
// Synthesizable — Behavioral 16-bit LFSR (maximal-length example)
module lfsr_16bit(
    input  clk, reset_n, enable,
    output reg [15:0] lfsr
);
    localparam SEED = 16'hACE1;      // any non-zero, non-all-ones seed
    wire feedback = lfsr[15] ^ lfsr[14] ^ lfsr[12] ^ lfsr[3]; // example tap set

    always @(posedge clk or negedge reset_n)
        if (!reset_n)     lfsr <= SEED;
        else if (enable)  lfsr <= {lfsr[14:0], feedback};
endmodule
```

```
LFSR conceptual topology (16-bit, 3-tap example):

 ┌────────────────────────────────────────────┐
 │   [15][14][13]...[3][2][1][0]  <- shift reg │
 │     │    │             │                     │
 │     └────┴──────XOR────┘── feedback bit      │
 │                  │                            │
 └──────────────────┴──> fed back into bit[0]────┘
```

---

## 12. Counters & Frequency Dividers

### Synchronous Counter
- "Synchronous" = all flip-flops share the same clock.
- Conceptually: an **N-bit register** + an **N-bit adder** feeding back `count + 1` into the register each clock.

```verilog
// Synthesizable — Behavioral, parameterized N-bit up counter
module counter_n #(parameter WIDTH = 3)(
    input  clk, reset_n,
    output reg [WIDTH-1:0] counter
);
    always @(posedge clk or negedge reset_n)
        if (!reset_n) counter <= {WIDTH{1'b0}};
        else          counter <= counter + 1'b1;   // wraps automatically on overflow
endmodule
```

### Up/Down Counter with Load
```verilog
// Synthesizable — Behavioral up/down counter with parallel load
module counter_up_down #(parameter WIDTH = 3)(
    input  clk, reset_n, load_enable, up_down,
    input  [WIDTH-1:0] counter_in,
    output reg [WIDTH-1:0] counter
);
    always @(posedge clk or negedge reset_n) begin
        if (!reset_n)          counter <= {WIDTH{1'b0}};
        else if (load_enable)  counter <= counter_in;
        else if (up_down)      counter <= counter + 1'b1;
        else                   counter <= counter - 1'b1;
    end
endmodule
```

### Frequency Divider
Formula: `f_out = f_in / N`, where `N = 2^n` requires `n` flip-flops (each successive flip-flop stage gives `f_in/2`, `f_in/4`, `f_in/8`, ...).

```verilog
// Synthesizable — Structural + Behavioral mix: divide-by-2 + N-bit counter
module clock_div_n #(parameter WIDTH = 4)(
    input  clk, reset_n,
    output reg clock_div2,
    output [WIDTH-1:0] counter
);
    always @(posedge clk or negedge reset_n)
        if (!reset_n) clock_div2 <= 1'b0;
        else          clock_div2 <= ~clock_div2;   // toggle each clock -> divide by 2

    counter_n #(.WIDTH(WIDTH)) count_inst (
        .clk(clk), .reset_n(reset_n), .counter(counter)
    );
    // counter[0] = f/2, counter[1] = f/4, ... counter[WIDTH-1] = f/2^WIDTH
endmodule
```

### Non-Power-of-2 Divider (Divide-by-3 Example)
Requires a **custom schematic** (not simply cascaded toggle flip-flops): 3 D flip-flops (2 positive-edge, 1 negative-edge), with feedback logic gating an AND/OR network to produce a 1.5-clock-cycle-wide pulse combined from two flip-flop outputs.

> [!NOTE]
> Divide-by-3, divide-by-5, divide-by-7 (odd ratios) each require **bespoke flip-flop + gate topologies** — unlike power-of-2 ratios which simply cascade toggle flip-flops from a counter's bit outputs.

---

## 13. Functions & Tasks

Both help avoid inline code duplication, improving scalability, maintainability, and reducing bugs — critical as circuit complexity grows (many I/O ports each requiring individually-written stimulus/checks would not scale).

### Verilog Functions
| Property | Detail |
|---|---|
| Time consumption | **Zero** — cannot contain `#`, `wait`, `@posedge`, etc. |
| Inputs | At least 1 (can have multiple) |
| Outputs | **None** — the function name itself *is* the return value |
| Can call | Other functions (not tasks — tasks may consume time) |
| Synthesizable | Yes (models combinational logic) |

```verilog
// Synthesizable — combinational function
module func_example;
    function [8:0] sum;    // 9-bit return value (8-bit + carry)
        input [7:0] a, b;
        begin
            sum = a + b;
        end
    endfunction
endmodule
```

### Recursive Functions
Require the `automatic` keyword — allocates a **unique stack frame per call** (vs. `static`, the Verilog default, where all calls share the same storage — unsafe for recursion).

```verilog
// Synthesizable/behavioral — recursive factorial function
function automatic integer factorial;
    input integer n;
    integer result;
    begin
        if (n == 0) result = 1;
        else        result = n * factorial(n - 1);
        factorial = result;
    end
endfunction

// Recursive Fibonacci function
function automatic integer fibonacci;
    input integer n;
    integer result;
    begin
        if      (n == 0) result = 0;
        else if (n == 1) result = 1;
        else              result = fibonacci(n-1) + fibonacci(n-2);
        fibonacci = result;
    end
endfunction
```

### Verilog Tasks
| Property | Detail |
|---|---|
| Time consumption | **Allowed** — can use `#`, `@(posedge clk)`, `wait`, etc. |
| Inputs / Outputs | Both allowed; also `inout` |
| Can call | Other tasks and functions |
| Synthesizable | **No** — used almost exclusively in testbenches |

```verilog
// Non-synthesizable — testbench task: write data to a register-modeling testbench
task write_data;
    input [7:0] _data;
    input       _load;
    begin
        @(posedge clk);
        load = _load;
        data = _data;
        @(posedge clk);
        load = 1'b0;
    end
endtask
```

> [!TIP]
> **Preferred task-declaration style** (per the instructor): declare all inputs/outputs in the task's parentheses (`task name(input a, output b);`) rather than inside the task body or relying purely on global variables — this style is clearer and easier to maintain.

---

## 14. Self-Checking Testbenches & Golden Models

As designs scale to **hundreds of I/O ports**, manual stimulus generation and eyeballing waveforms becomes impossible. The industry solution: **automatic, self-checking verification**.

### The Golden Model Concept
A **golden model** is an independently-coded, "unlocked"/reference behavioral model of the same functional spec, used to generate **expected** results for comparison against the DUT's **observed** results.

```
        ┌───────────────┐        ┌──────────────────┐
Stimulus│               │        │                   │
──────► │      DUT      │─out──► │                   │
   │    └───────────────┘        │   Compare Task    │──► PASS/FAIL
   │                              │  (===  equality)  │      report
   └──►┌───────────────┐         │                   │
       │  Golden Model │─exp───► │                   │
       │  (reference)  │         └───────────────────┘
       └───────────────┘
```

> [!NOTE]
> Best practice: the **DUT and golden model should be authored by different people** (ideally DUT by RTL designer, golden model by a verification engineer), both working strictly from the same functional specification document — this avoids both people making the *same* misinterpretation/mistake.

### Self-Checking Testbench Pattern (Register Example)
```verilog
// Non-synthesizable — self-checking testbench pattern
integer test_count, success_count, error_count;

task check_data;
    input [7:0] expected;
    input [7:0] observed;
    begin
        test_count = test_count + 1;
        if (observed === expected) begin
            success_count = success_count + 1;
            $display("PASS: expected=%h observed=%h", expected, observed);
        end else begin
            error_count = error_count + 1;
            $display("ERROR: expected=%h observed=%h", expected, observed);
        end
    end
endtask
```

> [!WARNING]
> Use the **case equality operator `===`** (not `==`) inside compare tasks — the DUT could legitimately output `x`/`z` on a bug, and `==` would return `x` (masking the failure) instead of a clean, countable mismatch.

### Worked Example — Pipe Register with Intentional Bug
The course deliberately breaks an 8-bit pipeline register's **MSB connectivity** (bit 7 of `d` never reaches bit 7 of `q`, staying stuck at the async-reset value of 0). A randomized self-checking testbench comparing `d` against `q` one clock later catches roughly **6/10 tests failing** — specifically every test vector where `d[7]=1`, since that's the only condition where the bug's effect is observable.

### Worked Example — ALU with Golden-Model-Based Randomized Verification
- The golden-model function computes the *correct* `OP_ADD` behavior including carry-out (`{carry_out, y} = a + b`), while the buggy DUT drops the carry (`y = a + b`).
- Running only **5 random tests** may miss the bug (overflow condition didn't happen to occur); running **100 tests** exposed ~3% failures; running **1000 tests** exposed ~5% failures.

> [!TIP]
> **This is the core argument for randomized, high-volume, self-checking verification over hand-written directed tests**: rare corner cases (like an 8-bit addition overflowing past 255) may simply never occur in a handful of manually-chosen test vectors, yet occur reliably given enough random iterations — and a self-checking testbench is what makes running thousands of iterations *actually useful* (a human cannot eyeball thousands of waveform cycles).

### Useful System Tasks for Randomization
- `$random` — returns a 32-bit signed pseudo-random number.
- `$urandom` — returns a 32-bit **unsigned** pseudo-random number (auto-truncated to destination width on assignment).
- `$urandom_range(min, max)` / `$random_range` (tool-dependent) — bounded random value.

---

## 15. Memory Design: SRAM / DRAM / ROM

### Fundamentals
| Term | Meaning |
|---|---|
| **RAM** | Random Access Memory — individual bit/word access, read+write capable, **volatile** |
| **SRAM** | Static RAM — built from flip-flops/latches; faster; simpler; more silicon area; no refresh needed |
| **DRAM** | Dynamic RAM — flip-flop + capacitor per bit; smaller area; requires **periodic refresh**; more power overhead from refresh; used for large, cheap main memory |
| **ROM** | Read-Only Memory — **non-volatile** (`EEPROM`, `Flash` variants included) |

**Sizing formula:** a memory with `n`-bit address bus can address `2^n` distinct locations; total memory size = `n × 2^n` bits (word-width × depth, more precisely `WIDTH × DEPTH`).

### Single-Port Asynchronous-Read SRAM
```verilog
// Synthesizable — Behavioral single-port SRAM, sync write / async read
module sram_async_read #(parameter WIDTH = 8, parameter DEPTH = 16)(
    input  clk,
    input  write_enable,
    input  [$clog2(DEPTH)-1:0] address,
    input  [WIDTH-1:0] data_in,
    output [WIDTH-1:0] data_out
);
    reg [WIDTH-1:0] ram [0:DEPTH-1];   // 2D array = memory model

    always @(posedge clk)
        if (write_enable) ram[address] <= data_in;

    assign data_out = ram[address];    // asynchronous, combinational read
endmodule
```
> [!NOTE]
> SRAM models have **no reset pin** in real silicon — the default simulation value is `x` until a location is written, which is realistic behavior (an untouched memory cell holds indeterminate data in hardware too).

### Single-Port Synchronous-Read SRAM
```verilog
// Synthesizable — Behavioral, sync write AND sync read (1-clock read latency)
module sram_sync_read #(parameter WIDTH = 8, parameter DEPTH = 16)(
    input  clk,
    input  write_enable,
    input  [$clog2(DEPTH)-1:0] address,
    input  [WIDTH-1:0] data_in,
    output reg [WIDTH-1:0] data_out
);
    reg [WIDTH-1:0] ram [0:DEPTH-1];
    reg [$clog2(DEPTH)-1:0] address_buf;

    always @(posedge clk) begin
        if (write_enable) ram[address] <= data_in;
        address_buf <= address;              // buffered address -> 1-cycle read latency
    end

    always @(*) data_out = ram[address_buf];
endmodule
```

### Dual-Port SRAM (Independent Read/Write Addresses)
```verilog
// Synthesizable — Parameterized dual-port, sync write / async read
module ram_async_dualport #(
    parameter WIDTH = 8,
    parameter DEPTH = 16,
    parameter ADDR_W = $clog2(DEPTH)
)(
    input  clk,
    input  write_enable,
    input  [ADDR_W-1:0] address_write, address_read,
    input  [WIDTH-1:0]  data_write,
    output [WIDTH-1:0]  data_read
);
    reg [WIDTH-1:0] ram [0:DEPTH-1];

    always @(posedge clk)
        if (write_enable) ram[address_write] <= data_write;

    assign data_read = ram[address_read];   // independent, async read port
endmodule
```
> [!NOTE]
> Dual-port SRAM enables **simultaneous read and write to different addresses** — a common building block for FIFOs, pipeline buffers, and cross-domain buffering.

### ROM (Loaded From a Hex File)
```verilog
// Synthesizable — synchronous ROM, contents loaded from an external .hex file
module rom_sync #(parameter WIDTH = 8, parameter DEPTH = 16)(
    input  clk,
    input  [$clog2(DEPTH)-1:0] address,
    output reg [WIDTH-1:0] data_out
);
    reg [WIDTH-1:0] rom [0:DEPTH-1];

    initial begin
        $readmemh("rom_init.hex", rom, 0, DEPTH-1);  // load Intel-hex-format init file
    end

    always @(posedge clk)
        data_out <= rom[address];
endmodule
```
- `$readmemh(filename, memory, start_addr, stop_addr)` — loads **hex**-formatted values.
- `$readmemb(...)` — equivalent for **binary**-formatted values.

---

## 16. Finite State Machines (FSM)

An FSM is a mathematical model processing an input sequence over time to produce deterministic outputs, based on a finite set of states.

### Mealy vs Moore

| Property | **Moore** | **Mealy** |
|---|---|---|
| Output depends on | Current **state only** | Current state **and** current inputs |
| Number of states | More (mnemonic: "Moore is more" states) | Fewer (mnemonic: "Mealy is less") |
| Output decode logic | More logic → larger delay | Less logic → faster reaction |
| Reaction to input change | Only after next state transition | Can react **within the same clock cycle** |
| Example | Modulo-N counter (e.g. filling a 100-pill bottle) | Metro turnstile, traffic-light controller |

> [!NOTE]
> **Historical context:** Moore and Mealy machines are named after **Edward F. Moore** and **George H. Mealy**, foundational figures in automata theory (1950s). This classification predates HDLs entirely and is core to all of digital sequential design, not just Verilog.

### The Recommended Mealy FSM Template (3-Block Structure)
```
┌─────────────────────────┐     ┌───────────────────┐     ┌───────────────────────┐
│  Next-State Combinational│────►│  State Register    │────►│ Output Combinational   │
│  Logic (always @*)       │     │ (always @(posedge  │     │ Logic (optional,       │
│  case(state) ... default │◄────│  clk or negedge    │     │ inputs + state)        │
└─────────────────────────┘     │  reset_n))         │     └───────────────────────┘
                                  └───────────────────┘
```

```verilog
// Synthesizable — Generic Mealy FSM template
module fsm_template(
    input  clk, reset_n,
    input  in,
    output reg out
);
    // 1. State encoding
    parameter S0 = 2'b00, S1 = 2'b01, S2 = 2'b10;

    reg [1:0] state, next_state;

    // 2. Next-state combinational logic — ALWAYS use always @(*)
    always @(*) begin
        next_state = state;   // default: hold current state (prevents latches)
        out = 1'b0;           // default output
        case (state)
            S0: if (in) next_state = S1;
            S1: if (in) next_state = S2; else next_state = S0;
            S2: begin next_state = S0; out = 1'b1; end
            default: next_state = S0;   // prevents lock-up in unreachable states
        endcase
    end

    // 3. State (sequential) register — identical pattern in every FSM
    always @(posedge clk or negedge reset_n)
        if (!reset_n) state <= S0;
        else          state <= next_state;
endmodule
```

> [!WARNING]
> **Critical FSM coding rules, repeated throughout the course:**
> 1. The next-state logic block must be `always @(*)` — never a partial sensitivity list.
> 2. Use `case(state)` — **never** `case(next_state)` — inside the next-state block.
> 3. Always include a `default:` case, even if all encoded values seem covered — it protects against corrupted/unreachable states (e.g. from SEU radiation events) and prevents synthesis from inferring an unwanted latch.
> 4. Assign **default values** for `next_state` and all Mealy outputs at the top of the block, before the `case`, so every signal is assigned on every path.
> 5. Use `parameter`/`localparam` (not `` `define ``) for state encodings — gives better synthesis tool recognition of FSM structure (many tools can auto-extract and display a state diagram).

### State Encoding Styles
| Style | Trade-off |
|---|---|
| Binary/sequential | Fewest flip-flops, more decode logic |
| **One-hot** | 1 flip-flop per state (more FFs), but very fast/simple decode logic — favored for high clock frequency; more invalid/unreachable states, hence `default:` is critical |
| Gray code | Only 1 bit changes per transition — lower switching power/noise, useful across clock domains |

### Worked Example 1 — Metro Turnstile (Moore-ish/Mealy hybrid, 3 states)
States: `IDLE` → (card inserted, `validate_code` set) → `CHECK_CODE` → (code valid, e.g. between 4–11) → `ACCESS_GRANTED` (door opens, waits 15 clocks) → back to `IDLE`. Invalid code returns directly to `IDLE`.

```
        validate_code=1               code valid (4..11)
 IDLE ───────────────────► CHECK_CODE ───────────────────► ACCESS_GRANTED
   ▲                             │                                │
   └─────────────────────────────┘ (code invalid)                 │
   ▲                                                               │
   └───────────────────────────(after 15 clocks, timer==15)───────┘
```

### Worked Example 2 — Traffic Light Semaphore (One-Hot Mealy)
States: `OFF → RED (50 clks) → YELLOW (10 clks) → GREEN (30 clks) → RED → ...`; any state returns to `OFF` immediately if `enable=0` (higher-priority transition).

```verilog
// Synthesizable — condensed semaphore FSM (one-hot encoding)
module semaphore_fsm(
    input  clk, reset_n, enable,
    output reg red, yellow, green
);
    parameter OFF=4'b0001, RED=4'b0010, YELLOW=4'b0100, GREEN=4'b1000;
    reg [3:0] state, next_state;
    reg [5:0] timer;
    reg timer_clear;

    always @(*) begin
        next_state  = OFF;      // default -> prevents lock-up
        timer_clear = 1'b0;
        red = 0; yellow = 0; green = 0;
        case (state)
            OFF:    if (enable) next_state = RED; else next_state = OFF;
            RED:    begin red = 1;
                     if (timer == 6'd50) begin next_state = YELLOW; timer_clear = 1; end
                     else next_state = RED; end
            YELLOW: begin yellow = 1;
                     if (timer == 6'd10) begin next_state = GREEN; timer_clear = 1; end
                     else next_state = YELLOW; end
            GREEN:  begin green = 1;
                     if (timer == 6'd30) begin next_state = RED; timer_clear = 1; end
                     else next_state = GREEN; end
            default: next_state = OFF;
        endcase
        if (!enable) next_state = OFF;   // higher-priority exit from any state
    end

    always @(posedge clk or negedge reset_n)
        if (!reset_n) state <= OFF;
        else          state <= next_state;

    always @(posedge clk or negedge reset_n)
        if (!reset_n || timer_clear || !enable) timer <= 6'd0;
        else if (state != OFF) timer <= timer + 1'b1;
endmodule
```
> [!TIP]
> **Area optimization technique used throughout the course:** compare the timer against a specific bit-width constant (`timer == 6'd50`, not `timer == 50`) to force synthesis of a compact N-bit equality comparator instead of a full 32-bit integer comparator. Similarly, prefer `==` equality comparisons over `<=`/`>=` when only an exact-value check is functionally needed — equality comparators synthesize smaller than magnitude comparators.

### Sequence Detector (Non-Overlapping vs Overlapping)
Detects the bit pattern `101` on a serial input `seq_in`, asserting `detected` for one clock.
- **Non-overlapping**: after detecting `101`, restart search fresh from state `S1` (assumes the last `1` cannot be reused as the start of a new match).
- **Overlapping**: after detecting `101`, return to state `S10` instead of `S1`, allowing the trailing `01` to potentially combine with new incoming bits to detect an overlapping match (e.g. detects both matches within `10101`).

```
Non-overlapping FSM:                     Overlapping FSM (only difference marked *):
   S1 --(seq=1)--> S1                       S1 --(seq=1)--> S1
   S1 --(seq=0)--> S10                      S1 --(seq=0)--> S10
   S10--(seq=1)--> S101 (detected=1)-->S1   S10--(seq=1)--> S101(detected=1)-->*S10*
   S10--(seq=0)--> S10                      S10--(seq=0)--> S10
   S101-->S1 (regardless of seq)            S101-->*S10* (regardless of seq)   *difference
```

---

## 17. FIFO Design

**FIFO** (First-In, First-Out) — a circular-buffer memory structure that preserves data arrival order; used for buffering, flow control, and clock-domain synchronization.

### Core Mechanism
- Two pointers: **write pointer** and **read pointer**, both are addresses into the memory array.
- Each read/write operation increments its respective pointer.
- **Empty condition**: `read_pointer == write_pointer`.
- **Full condition**: also `read_pointer == write_pointer` in the raw address bits — ambiguous unless resolved!

**Standard resolution: add one extra MSB bit to each pointer** ("wrap bit"), which toggles every time the pointer wraps around the memory depth:
| Condition | Test |
|---|---|
| **Empty** | `wrap_bit_read == wrap_bit_write` AND `read_pointer[N-1:0] == write_pointer[N-1:0]` |
| **Full** | `wrap_bit_read != wrap_bit_write` AND `read_pointer[N-1:0] == write_pointer[N-1:0]` |

### Synchronous vs Asynchronous FIFO
| Type | Read/write clock | Typical use |
|---|---|---|
| **Synchronous FIFO** | Same clock domain for read & write | Simple buffering within one clock domain |
| **Asynchronous FIFO** | Different clocks for read & write | **Clock Domain Crossing (CDC)** — transmitting data safely between two independently-clocked subsystems |

> [!WARNING]
> Asynchronous FIFOs (not built in this course, but referenced conceptually) require careful handling of **metastability** at the CDC boundary — typically via Gray-coded pointers and multi-stage synchronizer flip-flops, since a signal changing near a sampling flip-flop's setup/hold window can resolve to an unpredictable, potentially-metastable intermediate voltage state before settling to 0 or 1.

```verilog
// Synthesizable — 32-bit-wide x 8-location Synchronous FIFO
module fifo_sync #(parameter WIDTH = 32, parameter DEPTH = 8)(
    input  clk, reset_n,
    input  chip_select, write_enable, read_enable,
    input  [WIDTH-1:0] data_in,
    output reg [WIDTH-1:0] data_out,
    output full, empty
);
    localparam DEPTH_LOG = $clog2(DEPTH);

    reg [WIDTH-1:0] mem [0:DEPTH-1];
    reg [DEPTH_LOG:0] write_ptr, read_ptr;   // 1 extra bit for full/empty detection

    // Write pointer
    always @(posedge clk or negedge reset_n)
        if (!reset_n) write_ptr <= 0;
        else if (chip_select && write_enable && !full)
            write_ptr <= write_ptr + 1'b1;

    // Read pointer
    always @(posedge clk or negedge reset_n)
        if (!reset_n) read_ptr <= 0;
        else if (chip_select && read_enable && !empty)
            read_ptr <= read_ptr + 1'b1;

    assign empty = (write_ptr == read_ptr);
    assign full  = (write_ptr[DEPTH_LOG] != read_ptr[DEPTH_LOG]) &&
                   (write_ptr[DEPTH_LOG-1:0] == read_ptr[DEPTH_LOG-1:0]);

    // Memory write
    always @(posedge clk)
        if (chip_select && write_enable && !full)
            mem[write_ptr[DEPTH_LOG-1:0]] <= data_in;

    // Registered read (adds 1-cycle output buffer)
    always @(posedge clk)
        if (chip_select && read_enable && !empty)
            data_out <= mem[read_ptr[DEPTH_LOG-1:0]];
endmodule
```

```
FIFO conceptual layout (circular buffer view):

          write_ptr
              │
              ▼
   ┌───┬───┬───┬───┬───┬───┬───┬───┐
   │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │   (DEPTH = 8 locations)
   └───┴───┴───┴───┴───┴───┴───┴───┘
              ▲
              │
          read_ptr
```

> [!NOTE]
> **Write protection**: writes are ignored while `full=1` (data would be silently discarded — a data-loss condition the surrounding system must avoid by monitoring `full`). **Read behavior when empty**: continuing to assert `read_enable` while `empty=1` simply re-reads the last valid location without advancing the pointer — no new/garbage data is produced.

---

## 18. Capstone: Data-Transfer Bridge FSM

**Scenario:** Module 1 writes results into an 8-bit × 32-location SRAM. An FSM must **repackage** that data — combining two adjacent 8-bit bytes into one 16-bit word — and write it into a second, 16-bit × 16-location SRAM for Module 2 to consume.

### I/O Summary
| Port | Direction | Width | Purpose |
|---|---|---|---|
| `ram_in_write_enable` | in | 1 | Write enable for input (8-bit) SRAM |
| `ram_in_address_write` | in | 5 | Write address for input SRAM (32 locations) |
| `ram_in_data_write` | in | 8 | Write data for input SRAM |
| `opmode_in` | in | 1 | Pulse to trigger the transfer/resize process |
| `ram_out_address_read` | in | 4 | Read address for output (16-bit) SRAM |
| `ram_out_data_read` | out | 16 | Read data from output SRAM |
| `done_out` | out | 1 | Asserted when transfer completes; cleared by next `opmode_in` toggle |

### FSM States
```
 IDLE ──(opmode_in=1)──► READ_BYTE0 ──► READ_BYTE1 ──► WRITE_BYTE01 ──┐
   ▲                                                                   │
   └───────────(after all 32 input bytes processed)────────────────────┘
                (loops READ_BYTE0→READ_BYTE1→WRITE_BYTE01 16 times total)
```
- In `READ_BYTE0`/`READ_BYTE1`: update `ram_in_address_read` (via an incrementing "ram_pointer" counter).
- In `WRITE_BYTE01`: assert `ram_out_write_enable`; write address = `ram_pointer / 2`; write data = concatenation of the two buffered bytes.
- `done_out` sets (async, cleared by `opmode_in` toggle) when `ram_pointer` reaches its max value (31).

> [!NOTE]
> This exercise deliberately combines **structural** (instantiating two differently-sized SRAM sub-modules) with **behavioral** (the FSM's next-state and datapath logic) styles, plus **pipeline buffer registers** for the two read bytes — reinforcing that real designs freely mix all three coding styles as appropriate.

---

## 19. Data Encryption Module (Stream Cipher)

### Concept
- **Symmetric-key encryption**: same key used to encrypt and decrypt.
- **Stream cipher**: encrypts/decrypts **one byte (or bit) at a time** — course example references the **A5/1** GSM stream cipher algorithm.
- **Block cipher** (contrast): encrypts multiple bytes at once — course references **AES-128** (Advanced Encryption Standard).
- **Simplest stream-cipher technique**: XOR each plaintext character with a value from a **PRNG** (here, the LFSR from Section 11).

```
ENCRYPTION:                          DECRYPTION:
 seed ──► [PRNG] ──┐                  seed ──► [PRNG] ──┐
                    ├─XOR──► cipher                      ├─XOR──► plaintext
 plaintext ─────────┘         text     ciphertext ────────┘         (recovered)
```

> [!WARNING]
> Decryption **only works correctly if the same PRNG algorithm and the same seed** are used as during encryption — any mismatch produces unintelligible output, by design.

```verilog
// Synthesizable — 8-bit PRNG core for the stream cipher
module prng_8bit(
    input  clk, reset_n, load_seed, encrypt_enable,
    input  [7:0] seed_in,
    output reg [7:0] prng_reg
);
    parameter SEED = 8'hCD;   // must be non-zero, non-all-ones
    wire feedback = prng_reg[7] ^ prng_reg[5] ^ prng_reg[4] ^ prng_reg[3]; // example taps

    always @(posedge clk or negedge reset_n) begin
        if (!reset_n)             prng_reg <= SEED;
        else if (load_seed)       prng_reg <= seed_in;
        else if (encrypt_enable)  prng_reg <= {prng_reg[6:0], feedback};
    end
endmodule
```

```verilog
// Synthesizable — Top-level stream cipher encrypt/decrypt module
module top_encrypt(
    input  clk, reset_n, load_seed, encrypt_enable,
    input  [7:0] seed_in, data_in,
    output reg [7:0] data_out
);
    wire [7:0] prng_val;
    reg  [7:0] data_in_delayed;
    reg        encrypt_enable_delayed;

    prng_8bit prng_inst(
        .clk(clk), .reset_n(reset_n), .load_seed(load_seed),
        .encrypt_enable(encrypt_enable), .seed_in(seed_in), .prng_reg(prng_val)
    );

    // 1-cycle buffer to align data_in with the PRNG's registered output
    always @(posedge clk or negedge reset_n)
        if (!reset_n) begin
            data_in_delayed        <= 8'b0;
            encrypt_enable_delayed <= 1'b0;
        end else begin
            data_in_delayed        <= data_in;
            encrypt_enable_delayed <= encrypt_enable;
        end

    // XOR cipher stage
    always @(posedge clk or negedge reset_n)
        if (!reset_n)                       data_out <= 8'b0;
        else if (encrypt_enable_delayed)     data_out <= prng_val ^ data_in_delayed;
endmodule
```

> [!TIP]
> `encrypt_enable` gates the PRNG's shift so a **new pseudo-random value is generated only when a new character actually needs encrypting** — avoiding wasted/unsynchronized PRNG advances if characters arrive with variable timing gaps.

### Verification Flow for the Cipher
1. Instantiate both the RTL (`top_encrypt`) and a **golden model** (`top_encrypt_golden`, same spec, independently coded, e.g. grouping the feedback into a function) with identical stimulus.
2. Encrypt a test string (e.g., `"Red apple"`) character-by-character; compare DUT vs golden-model output each cycle using `===`.
3. **Reload the same seed** before attempting to decrypt (the PRNG's internal state has advanced past its initial seed after encryption — decryption must restart the keystream from the identical seed value used originally).
4. Decrypt the previously-encrypted ciphertext and confirm the recovered plaintext matches the original message.
5. View `data_in`/`data_out`/`data_out_ref` in the waveform viewer with **Radix → ASCII** to visually confirm: readable plaintext → unintelligible ciphertext → readable plaintext again.

---

## 20. Keyword / System Task / Operator Glossary

### Declaration & Structural Keywords
| Keyword | Meaning |
|---|---|
| `module` / `endmodule` | Defines a design unit's boundary |
| `input` / `output` / `inout` | Port directions |
| `wire` | Net type — driven by continuous assignment or gate/primitive output |
| `reg` | Variable type — driven inside a procedural block (`initial`/`always`) |
| `integer` | 32-bit signed variable |
| `real` | 64-bit floating point variable |
| `time` | 32-bit (unsigned) simulation-time variable |
| `parameter` | Compile-time constant, overridable per instantiation |
| `localparam` | Compile-time constant, **not** overridable from outside the module |
| `function` / `endfunction` | Zero-time combinational procedure returning a single value |
| `task` / `endtask` | Procedure that may consume time, with multiple I/O |
| `automatic` | Marks a function/task as **re-entrant** (own stack frame per call) — required for recursion |
| `initial` | Procedural block executed once, starting at time 0 |
| `always` | Procedural block that re-executes forever, triggered by its sensitivity list |
| `begin` / `end` | Groups multiple sequential statements (like `{ }` in C) |
| `case` / `casez` / `casex` / `endcase` | Multi-way branch construct |
| `if` / `else` | Conditional branch |
| `for` / `while` / `repeat` / `forever` | Looping constructs (mainly for testbenches/functions) |
| `assign` | Continuous assignment (combinational, drives a net) |
| `posedge` / `negedge` | Edge-detection qualifiers for sensitivity lists |
| `or` (in sensitivity list) | Separates multiple signals: `@(a or b or c)` |
| `@(*)` / `always @*` | Wildcard sensitivity list — auto-includes every signal read on the RHS |
| `generate` / `endgenerate` | (Verilog-2001+) Conditional/looped hardware generation at elaboration time |

### System Tasks & Functions (all prefixed with `$`)
| Task | Purpose |
|---|---|
| `$display` | Print a message once (like `printf`, auto-newline) |
| `$write` | Like `$display` but no auto-newline |
| `$monitor` | Print automatically **whenever any listed variable changes** |
| `$time` | Returns current simulation time |
| `$finish` | Ends the simulation |
| `$stop` | Suspends simulation (can resume, e.g. in interactive debug) |
| `$random` | Returns a 32-bit signed pseudo-random number |
| `$urandom` | Returns a 32-bit unsigned pseudo-random number |
| `$urandom_range(min,max)` | Bounded unsigned random number |
| `$readmemh` / `$readmemb` | Load memory array contents from a hex/binary text file |
| `$log2` (informal, `$clog2` in later standards) | Ceiling of base-2 logarithm — used to size address buses from a depth parameter |

### Operator Quick Reference
| Symbol(s) | Category | Meaning |
|---|---|---|
| `&`, `\|`, `^`, `~&`, `~\|`, `~^`/`^~`, `~` | Bitwise | Per-bit AND/OR/XOR/NAND/NOR/XNOR/NOT |
| `&&`, `\|\|`, `!` | Logical | AND/OR/NOT returning single-bit true/false/x |
| `+ - * / %` `**` | Arithmetic | Standard math ops (`**` = exponentiation) |
| `<< >>` `<<< >>>` | Shift | Logical left/right; arithmetic left/right (sign-preserving) |
| `< > <= >=` | Relational | Less/greater/less-eq/greater-eq |
| `== !=` | Logical equality | Design-time equality (x/z → x result) |
| `=== !==` | Case equality | Verification-time equality (exact x/z match, always 0/1) |
| `?:` | Conditional (ternary) | `cond ? if_true : if_false` |
| `{ }` | Concatenation | Joins bit-fields into a wider vector |
| `{N{ }}` | Replication | Repeats a value N times |
| `=` | Blocking assignment | Sequential/immediate — use for **combinational** logic |
| `<=` | Non-blocking assignment | Scheduled/deferred — use for **sequential** logic |
| `#` | Delay control | `#5 a = b;` — delays 5 time units (simulation only, non-synthesizable) |

---

## 21. Acronym Glossary

| Acronym | Expansion | Notes |
|---|---|---|
| **HDL** | Hardware Description Language | Verilog, VHDL, SystemVerilog |
| **RTL** | Register Transfer Level | The abstraction level where most synthesizable Verilog is written |
| **ASIC** | Application-Specific Integrated Circuit | Custom-fabricated chip, not reconfigurable post-manufacture |
| **FPGA** | Field-Programmable Gate Array | Reconfigurable chip, reprogrammed after manufacture |
| **DUT** | Design Under Test | The module being verified by a testbench |
| **FSM** | Finite State Machine | Sequential circuit with a finite number of defined states |
| **SRAM** | Static Random Access Memory | Flip-flop/latch-based, faster, no refresh needed |
| **DRAM** | Dynamic Random Access Memory | Capacitor-based, denser, requires refresh |
| **ROM** | Read-Only Memory | Non-volatile; includes EEPROM, Flash |
| **FIFO** | First-In, First-Out | Order-preserving buffer memory structure |
| **LRM** | Language Reference Manual | The formal specification document for a language (e.g. the IEEE 1364 LRM for Verilog) |
| **EDA** | Electronic Design Automation | Software tooling for chip/PCB design |
| **CAD** | Computer-Aided Design | Schematic/layout creation tools |
| **CAE** | Computer-Aided Engineering | Schematic/behavior analysis tools |
| **PRNG** | Pseudo-Random Number Generator | Deterministic, seed-based random sequence generator (e.g. LFSR) |
| **LFSR** | Linear Feedback Shift Register | Shift register + XOR feedback taps; used as a PRNG |
| **ALU** | Arithmetic Logic Unit | Combinational block performing arithmetic + bitwise ops |
| **BCD** | Binary-Coded Decimal | Each decimal digit encoded in its own 4-bit nibble |
| **AES** | Advanced Encryption Standard | 128-bit (or larger) symmetric block cipher |
| **CDC** | Clock Domain Crossing | Signal/data transfer between differently-clocked logic domains |
| **PIPO / SISO / PISO / SIPO** | Parallel/Serial In, Parallel/Serial Out | Shift-register data-movement classifications |
| **MSB / LSB** | Most/Least Significant Bit | Highest/lowest weighted bit in a vector |
| **SoC** | System on Chip | A chip integrating CPU, memory, peripherals, etc. on one die |
| **PCB** | Printed Circuit Board | Physical board interconnecting multiple chips/components |

---

## 22. Deep Dive: Simulation Event Scheduling

The course's testbenches rely on Verilog's discrete-event simulation model. Here's what happens "under the hood" in a simulator like ModelSim/Questa (this section extends beyond what the transcript states explicitly, for completeness):

### The Stratified Event Queue
At each simulation time step, the event queue is logically divided into **regions**, processed in this order:
1. **Active region** — blocking assignments (`=`), continuous assignments (`assign`), `$display`, evaluation of RHS of non-blocking assignments.
2. **Inactive region** — statements scheduled with a zero-delay (`#0`) from the active region.
3. **NBA (Non-Blocking Assign) update region** — the **actual updates** of non-blocking assignment (`<=`) targets happen here, *after* all active-region RHS evaluations for the current time step have completed.
4. **Postponed region** — `$monitor`/`$strobe` evaluations, guaranteed to see the final, settled values for the time step.

> [!NOTE]
> **Why non-blocking (`<=`) is mandatory for sequential logic:** because all `<=` RHS values are evaluated in the Active region using the **old** register values, and the actual register updates are deferred to the NBA region — every flip-flop in an `always @(posedge clk)` block sees the same, consistent "before this edge" values of every other flip-flop it depends on. This correctly models real parallel hardware flip-flops all sampling simultaneously on a clock edge. Using blocking (`=`) here instead can create **simulation race conditions** where the result depends on the arbitrary order the simulator happens to evaluate multiple always blocks, potentially not matching real hardware behavior at all.

### Race Condition Example (why the ordering rules matter)
```verilog
// PROBLEMATIC: blocking assignment for sequential logic can create simulator-order-dependent bugs
always @(posedge clk) a = b;      // block 1
always @(posedge clk) c = a;      // block 2 -- depends on execution ORDER of block 1 vs 2!
```
If block 2 executes before block 1 at a given edge, `c` gets the **old** value of `a`; if block 1 executes first, `c` gets the **new** value of `a` — this is a real race condition in the LRM's execution model, and different simulators (or even the same simulator on different runs) are not required to resolve it identically. Using `<=` for both fixes this deterministically, because all RHS evaluations happen in the Active region against pre-edge values, before any NBA updates commit.

### What Synthesis Actually Does With Your Code
When RTL is synthesized:
1. **Technology-independent optimization**: the front-end tool converts your behavioral/dataflow/structural Verilog into a generic Boolean/register netlist, applying logic optimizations (Boolean minimization, common sub-expression elimination, resource sharing/muxing of arithmetic units where legal).
2. **Technology mapping**: the generic netlist is mapped onto the **target technology library** — standard cells (AND2X1, NAND2X2, DFFX1, etc.) for an ASIC flow, or **LUTs (Look-Up Tables) + dedicated hard blocks** (carry chains, embedded RAM blocks, DSP slices) for an FPGA flow.
3. **Placement & routing**: physical placement of cells/LUTs on the die and routing of interconnect — this is where actual propagation delays, wire capacitance, and timing closure become concrete numbers (static timing analysis, STA).
4. Each `always @(posedge clk)` block with a `reg` typically becomes one or more **flip-flops**; each combinational `always @(*)`/`assign` becomes a network of gates or LUTs; a `reg [W-1:0] mem [0:D-1]` array is typically inferred as a **Block RAM (BRAM)** primitive on FPGAs (rather than discrete flip-flops) if it's large enough and structured in a recognized RAM-inference pattern.

### Area vs Speed Trade-offs (recap of course's explicit and implicit lessons)
| Technique | Area impact | Speed impact |
|---|---|---|
| One-hot state encoding | More flip-flops (+area) | Faster state decode (+speed) |
| Binary state encoding | Fewer flip-flops (−area) | More decode logic (−speed) |
| Equality (`==`) vs magnitude (`<`,`>`) comparators | Equality is smaller | Magnitude comparators are typically slower/larger for wide buses |
| Sized literals (`1'b1`) vs bare integers (`1`) in +/- ops | Sized avoids inferring 32-bit integer arithmetic (−area) | N/A |
| Pipelining (extra pipeline registers) | More flip-flops (+area) | Shorter combinational paths → higher max clock frequency (+speed), at the cost of added latency (cycles) |
| Latches vs Flip-flops | Latches ~half the area | Flip-flops are more robust/standard for synchronous design |

---

## 23. Cheat Sheet: Rules of Thumb

| Rule | Why |
|---|---|
| Combinational logic → `always @(*)` + blocking `=` | Matches simulation to real parallel-gate hardware behavior; prevents stale-value bugs |
| Sequential logic → `always @(posedge clk)` + non-blocking `<=` | Prevents simulation race conditions; correctly models simultaneous flip-flop sampling |
| Always include `default:` in every `case` statement | Prevents unintended latch inference and FSM lock-up in unreachable states |
| Assign default values at the top of a combinational block | Guarantees every signal is driven on every code path (no latches) |
| LHS of `assign` (continuous assignment) must be `wire`, never `reg` | Continuous assignments drive nets, not procedural variables |
| LHS of an `always` block driving sequential/combinational logic must be `reg` (or `wire` if purely structural gate output) | Procedural-block-driven signals require variable storage semantics |
| Use `1'b1` instead of bare `1` in +/− expressions on sized signals | Prevents accidental 32-bit integer-width synthesis/simulation mismatch |
| Use `===`/`!==` in testbench compare logic, `==`/`!=` in RTL | Testbenches need to catch `x`/`z` bugs explicitly; RTL equality naturally propagates `x` when appropriate |
| Reset signal in sensitivity list (`or negedge reset_n`) = asynchronous; omitted = synchronous | Fundamental, easily-overlooked semantic difference with real timing implications |
| Always parenthesize mixed-precedence expressions | Prevents subtle logic bugs and improves readability/maintainability |
| Use `parameter`/`localparam`, not `` `define ``, for FSM state encoding | Better synthesis tool recognition (auto state-diagram extraction) and cleaner scoping |
| A `function` cannot consume time or call a `task`; a `task` can do both | Functions model pure combinational logic; tasks are for testbench sequencing |
| Recursive functions require the `automatic` keyword | Guarantees a unique stack frame per call (re-entrancy) |
| Self-checking testbenches + golden models + randomization > hand-written directed tests for finding rare bugs | Rare corner cases (e.g. arithmetic overflow) may never appear in a handful of manual vectors but reliably appear over many random iterations |
| A memory array (`reg [W-1:0] mem [0:D-1]`) has no reset — starts as `x` | Realistic model of untouched hardware memory cells |
| FIFO full/empty detection needs one extra pointer bit ("wrap bit") | Resolves the ambiguity that both conditions otherwise share the same raw pointer-equality signature |
| Synchronous reset waits for a clock edge; asynchronous reset acts immediately | Choose based on whether the design must be resettable even without a running clock |

---

*End of study notes — compiled from the Udemy course "Verilog HDL Fundamentals for Digital Design and Functional Verification" by Ovidiu Plugariu.*
