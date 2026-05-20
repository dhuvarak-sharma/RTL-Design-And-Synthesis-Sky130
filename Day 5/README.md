# Day 5 — If/Case Hazards, For Loops, and Generate Blocks

## Theory Topics Covered
---

### Inferred Latches

An inferred latch is unintentionally generated when all output conditions are not defined inside a combinational always block.

Latches can create:

- Timing issues
- Unpredictable behavior
- Synthesis mismatches

The workshop demonstrated how incomplete coding styles accidentally infer latches.

---

### For Loops

For loops help reduce repetitive Verilog code and improve scalability for large hardware structures.

They are especially useful in:

- Multiplexers
- Demultiplexers
- Bitwise operations
- Array-based hardware structures

---

### Generate Blocks

Generate blocks are used for repetitive hardware instantiation during synthesis. They are commonly used when multiple identical hardware blocks are required.

Generate blocks are especially useful in:

- Ripple Carry Adders
- Register Arrays
- Large Arithmetic Structures

---

### Ripple Carry Adders

A Ripple Carry Adder (RCA) is a combinational arithmetic circuit formed by connecting multiple full adders in series.

Each carry output propagates into the next stage, creating a ripple effect through the circuit.

Ripple Carry Adders are simple and area-efficient but may become slower for large bit-widths due to carry propagation delay.

---
---

## Labs

Day 5 covered two major topics. The first was understanding how poorly written `if` and `case` statements can cause **unintended latch inference** — a common mistake for beginners. The second was learning how to use **for loops** and **generate blocks** to write scalable, efficient hardware descriptions.

---

## Part 1 — If and Case Statements and Their Impacts on Hardware

### What is Latch Inference?

In hardware, a **latch** is a storage element that holds its output value when no new input condition is being applied to it. If we write a combinational `always` block that doesn't cover all possible input conditions, the synthesis tool has no choice but to infer a latch to "remember" the previous output. This is almost always unintentional and leads to bugs.

---

### Incomplete If Statement

```verilog
module incomp_if (input i0, input i1, input i2, output reg y);
  always @(*) begin
    if (i0)
      y <= i1;
  end
endmodule
```

Here, the `else` statement is not given. Thus, **inferred latching** takes place, and the system puts the value of `y` as it is for the else condition. The design acts like a D-Latch. Let us view the issues now:

![Waveform of incomp_if — latch behaviour visible](images/image43.png)

We can see that whenever `i0` goes low, `y` is latching onto its previous value.

![Zoomed waveform of incomp_if — latching confirmed](images/image44.png)

From this zoomed-in view, we can see that `y` latches onto its previous value. Our aim was to code a MUX, but instead we got a D-Latch.

Let us synthesize this code now:

```bash
yosys
read_liberty -lib /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_if.v
synth -top incomp_if
abc -liberty /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![Synthesis result of incomp_if — D-Latch inferred](images/image45.png)

Clearly, the synthesis tool has also inferred a D-Latch, as we expected.

---

### Incomplete If-Else If Statement

```verilog
module incomp_if2 (input i0, input i1, input i2, input i3, output reg y);
  always @(*) begin
    if (i0)
      y <= i1;
    else if (i2)
      y <= i3;
  end
endmodule
```

The code has an incomplete if-else structure because there is no final `else` condition. This causes latch inference, since `y` must store its old value when both conditions are false.

Let us view the waveform of this code:

![Waveform of incomp_if2 — latch behaviour visible](images/image46.png)

Whenever `i0` is high, `y` follows `i1`. Whenever `i0` is low and `i2` is high, `y` follows `i3`. However, whenever both `i0` and `i2` are low, output `y` is constant. This is because `y` latches onto its previous value till either `i0` or `i2` becomes high. This waveform thus shows that this design also behaves like a D-Latch. Let us confirm by synthesizing the code.

![Synthesis result of incomp_if2 — D-Latch inferred again](images/image47.png)

Again, a D-Latch has been inferred by the tool.

---

### Complete Case Statement

```verilog
module comp_case (input i0, input i1, input i2, input [1:0] sel, output reg y);
  always @(*) begin
    case(sel)
      2'b00 : y = i0;
      2'b01 : y = i1;
      default : y = i2;
    endcase
  end
endmodule
```

This code behaves like a multiplexer. There is no error in this code. The `default` statement makes the case statement complete, so no latch is inferred. Let us view the waveform:

![Waveform of comp_case — correct MUX behaviour](images/image48.png)

From the waveform, we can see that the code behaves like a multiplexer, as expected. There is no inferred latching. Thus, there are no issues in the code.

![Synthesis result of comp_case — MUX confirmed](images/image49.png)

The synthesis also confirms that there is MUX-like behaviour.

---

### Bad Case Statement

```verilog
module bad_case (
  input i0, input i1, input i2, input i3,
  input [1:0] sel,
  output reg y
);
  always @(*) begin
    case(sel)
      2'b00: y = i0;
      2'b01: y = i1;
      2'b10: y = i2;
      2'b1?: y = i3;
    endcase
  end
endmodule
```

From the code, we can infer that it is trying to be a MUX. However, due to the wildcard matching (`?`), the code loses its MUX-like behaviour as any value can come in place of the `?`. Thus, ambiguity, synthesis-simulation mismatches, and unintended latch behaviour may occur. Let us view the waveform:

![Waveform of bad_case — latch behaviour when sel=11](images/image50.png)

Here, we can see that the output signal `y` latches onto its previous value and is constant when the `sel` signal is `11`. There is ambiguity about why such behaviour happens. Let us see what the synthesis tool infers:

![Synthesis result of bad_case — mux inferred, mismatch with simulation](images/image51.png)

Here, we can see that the tool has inferred a MUX, but we detected latch-like behaviour in the simulation. Thus, there is also a synthesis-simulation mismatch, as we expected.

If we perform GLS on this code, the error and latching will get fixed:

![GLS waveform of bad_case — latch behaviour removed](images/image52.png)

We can see that now when the `sel` signal is `11`, the output is still being toggled with respect to `i3`. Thus, the latch-like behaviour has been removed, and the system does not get confused while selecting the value of `y`.

---

### Incomplete Case Statement

```verilog
module incomp_case (
  input i0, input i1, input i2,
  input [1:0] sel,
  output reg y
);
  always @(*) begin
    case(sel)
      2'b00 : y = i0;
      2'b01 : y = i1;
    endcase
  end
endmodule
```

From the code, we can infer that there will be latch inference for `sel = 10` or `11`, and thus `y` retains its previous value. Let us view the waveform now:

![Waveform of incomp_case — latch behaviour for sel=10,11](images/image53.png)

In the encircled region, the value of `y` is constant, and is equal to the previous value of `y` (which was low). Thus, we can see the latch-like behaviour occurring here as well.

![Synthesis result of incomp_case — D-Latch inferred](images/image54.png)

Even the synthesis tool is showing us that the D-Latch is influencing the output of `y`, as we expected.

> **Golden Rule:** In combinational `always` blocks, always cover every possible input condition — either with a `default` in `case` statements or with a final `else` in `if` chains. If you don't need a particular output for certain inputs, assign `y = 0` or any defined value to keep things explicit.

---

## Part 2 — For Loops and Generate Blocks

---

### 4:1 MUX Using For Loops

```verilog
module mux_generate (
  input i0, input i1, input i2, input i3,
  input [1:0] sel,
  output reg y
);
  wire [3:0] i_int;
  assign i_int = {i3, i2, i1, i0};
  integer k;
  always @(*) begin
    for (k = 0; k < 4; k = k + 1) begin
      if (k == sel)
        y = i_int[k];
    end
  end
endmodule
```

This code implements a 4-input and 1-output (4:1) multiplexer using a `for` loop, where `sel` selects one of the four inputs (`i0`–`i3`) and assigns it to `y`. Let us view the waveform:

![Waveform of mux_generate — correct MUX behaviour](images/image55.png)

We can see that the `sel` signal selects different values for `y` one by one, just as we expected.

---

### 1:8 Demultiplexer Using Case

```verilog
module demux_case (
  output o0, output o1, output o2, output o3,
  output o4, output o5, output o6, output o7,
  input [2:0] sel,
  input i
);
  reg [7:0] y_int;
  assign {o7, o6, o5, o4, o3, o2, o1, o0} = y_int;
  always @(*) begin
    y_int = 8'b0;
    case(sel)
      3'b000 : y_int[0] = i;
      3'b001 : y_int[1] = i;
      3'b010 : y_int[2] = i;
      3'b011 : y_int[3] = i;
      3'b100 : y_int[4] = i;
      3'b101 : y_int[5] = i;
      3'b110 : y_int[6] = i;
      3'b111 : y_int[7] = i;
    endcase
  end
endmodule
```

This is an example of a 1:8 demultiplexer where the input `i` is routed to one output (`o0`–`o7`) based on the value of `sel`. Let us view the waveform now:

![Waveform of demux_case — demux behaviour confirmed](images/image56.png)

From the waveform, it is evident that as `sel` changes, although the input is constant, the output keeps switching from `o0` to `o7`. Thus, the presence of a demux is confirmed.

However, this way of coding is inefficient if there are more outputs to be toggled to, such as a 1:32 demux or a 1:256 demux. Instead of using case statements, we can use a `for` loop. We will see that in the next lab.

---

### 1:8 Demultiplexer Using For Loops

```verilog
module demux_generate (
  output o0, output o1, output o2, output o3,
  output o4, output o5, output o6, output o7,
  input [2:0] sel,
  input i
);
  reg [7:0] y_int;
  assign {o7, o6, o5, o4, o3, o2, o1, o0} = y_int;
  integer k;
  always @(*) begin
    y_int = 8'b0;
    for (k = 0; k < 8; k = k + 1) begin
      if (k == sel)
        y_int[k] = i;
    end
  end
endmodule
```

This code also has the same functionality as the previous code — implementing a 1:8 demultiplexer. Let us view its waveform:

![Waveform of demux_generate — same output as demux_case](images/image57.png)

Clearly, the output is the same as the previous code. However, the number of lines needed to type this code is much less than that of the case statement. Using `for` loops makes multi-bit input or output codes very efficient.

---

### Ripple Carry Adder Using a Generate Block

A **Ripple Carry Adder (RCA)** chains multiple full adders together, where each stage's carry output feeds into the next stage's carry input.

**Full Adder Module:**
```verilog
module fa (input a, input b, input c, output co, output sum);
  assign {co, sum} = a + b + c;
endmodule
```

**8-Bit RCA Using Generate:**
```verilog
module rca (
  input [7:0] num1,
  input [7:0] num2,
  output [8:0] sum
);
  wire [7:0] int_sum;
  wire [7:0] int_co;
  genvar i;
  generate
    for (i = 1; i < 8; i = i + 1) begin
      fa u_fa_1 (.a(num1[i]), .b(num2[i]), .c(int_co[i-1]), .co(int_co[i]), .sum(int_sum[i]));
    end
  endgenerate
  fa u_fa_0 (.a(num1[0]), .b(num2[0]), .c(1'b0), .co(int_co[0]), .sum(int_sum[0]));
  assign sum[7:0] = int_sum;
  assign sum[8] = int_co[7];
endmodule
```

This code implements an 8-bit Ripple Carry Adder using multiple full adders connected in series, where each carry output propagates to the next stage.

> **Key Difference — `generate` vs `for` inside `always`:**
> - A `for` loop inside an `always` block runs at **simulation time** to evaluate logic
> - A `generate` block instantiates **hardware modules** at **elaboration time** — it physically creates multiple copies of a module and wires them together

While compiling, it is necessary to also include the full adder module in the iverilog statement:

```bash
iverilog fa.v rca.v tb_rca.v
./a.out
gtkwave tb_rca.vcd
```

![Waveform of rca — num1 + num2 = sum verified](images/image58.png)

Clearly, we can see that `num1 + num2 = sum`, as it is in the code. For example: `num1 = 244`, `num2 = 15`, `sum = num1 + num2 = 259`. This is true at any other point in the waveform as well — exactly as we expected from the code.

---

## Key Takeaways from Day 5

- Always cover every condition in `if`/`case` blocks inside `always @(*)` to avoid unintended latch inference
- Use `default` in `case` statements as a safety net
- Avoid wildcard `?` patterns in `case` — they introduce ambiguity and simulation-synthesis mismatches
- `for` loops inside `always` blocks create scalable combinational/sequential logic
- `generate` blocks instantiate multiple hardware modules — useful for arrays of components like adder chains

---

[← Day 4](./Day4.md) | [Back to Main README](./README.md)
