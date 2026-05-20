# Day 3 — Logic Optimization

## Theory Topics Covered

### Constant Propagation

Constant propagation is an optimization technique where synthesis tools replace signals with constant values whenever possible. If a signal is always tied to logic 0 or logic 1, unnecessary hardware can be removed.

This reduces:

- Gate count
- Area usage
- Power consumption

---

### State Optimization

State optimization simplifies sequential logic by reducing unnecessary states and minimizing hardware complexity.

Efficient state optimization improves:

- Circuit speed
- Area efficiency
- Power efficiency

---

### Cloning (Introduction)

Cloning refers to duplicating logic paths to reduce fanout and improve timing performance.

Although discussed briefly, it is an important optimization strategy used in larger digital systems.

---

### Retiming (Introduction)

Retiming is a sequential optimization technique where flip-flops are repositioned across combinational logic to improve timing performance without changing functionality.

Retiming helps balance logic delays and improve maximum clock frequency.

---

### Logic Simplification

Logic simplification reduces complex Boolean expressions into smaller and more efficient hardware structures.

Synthesis tools automatically simplify logic using Boolean algebra and optimization algorithms.

---
---

## Labs

On Day 3, we learned how Yosys can automatically simplify (optimize) logic designs to use fewer gates than a naive implementation would require. We explored both **combinational** and **sequential** optimization through seven example codes.

The key Yosys command used throughout this day is:

```bash
opt_clean -purge
```

This command tells Yosys to sweep through the design and remove any redundant logic or unused signals before mapping to gates.

---

## Combinational Logic Optimizations

---

### Code 1 — AND Gate Optimization

```verilog
module opt_check (input a, input b, output y);
  assign y = a ? b : 0;
endmodule
```

**Analysis:** If `a = 1`, output is `b`. If `a = 0`, output is `0`. This is exactly the behavior of `y = a & b` (AND gate).

**Synthesis Commands:**
```bash
yosys
read_liberty -lib /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check.v
synth -top opt_check
opt_clean -purge
abc -liberty /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![opt_check synthesis result — AND gate](images/image28.png)

We expected an AND gate, and we got an AND gate after optimization. Thus, optimization was successful.

---

### Code 2 — OR Gate Optimization

```verilog
module opt_check2 (input a, input b, output y);
  assign y = a ? 1 : b;
endmodule
```

**Analysis:** If `a = 1`, output is forced to `1`. If `a = 0`, output is `b`. This simplifies logically to `y = a | b` (OR gate).

**Synthesis Commands:**
```bash
yosys
read_liberty -lib /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check2.v
synth -top opt_check2
opt_clean -purge
abc -liberty /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![opt_check2 synthesis result — OR gate](images/image29.png)

We got an OR gate as expected, thus optimization was successful.

---

### Code 3 — 3-Input AND Gate Optimization

```verilog
module opt_check3 (input a, input b, input c, output y);
  assign y = a ? (c ? b : 0) : 0;
endmodule
```

**Analysis:** Using nested ternary operators:
- If `a = 0` → `y = 0`
- If `a = 1` and `c = 0` → `y = 0`
- If `a = 1` and `c = 1` → `y = b`

This is equivalent to `y = a & b & c` (3-input AND gate).

**Synthesis Commands:**
```bash
yosys
read_liberty -lib /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check3.v
synth -top opt_check3
opt_clean -purge
abc -liberty /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![opt_check3 synthesis result — 3-input AND gate](images/image30.png)

We got a 3-input AND gate as we expected, thus optimization was successful.

---

### Code 4 — XNOR Gate Optimization

```verilog
module opt_check4 (input a, input b, input c, output y);
  assign y = a ? (b ? (a & c) : c) : (!c);
endmodule
```

**Analysis:** This is a nested MUX-based logic circuit. If `a = 1`, the output depends on `b`: when `b = 1`, `y = a & c`; otherwise `y = c`. If `a = 0`, the output becomes `!c`. Despite the apparent complexity, this simplifies down to an XNOR gate, and the output does not depend on `b` at all.

**Synthesis Commands:**
```bash
yosys
read_liberty -lib /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check4.v
synth -top opt_check4
opt_clean -purge
abc -liberty /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![opt_check4 synthesis result — XNOR gate](images/image31.png)

The optimization shows that `opt_check4` simplifies down to an XNOR gate, and that the final output does not depend on the `b` pin.

---

### Code 5 — Multi-Module Optimization (AND + OR)

```verilog
module sub_module1(input a, input b, output y);
  assign y = a & b;
endmodule

module sub_module2(input a, input b, output y);
  assign y = a ^ b;
endmodule

module multiple_module_opt(input a, input b, input c, input d, output y);
  wire n1, n2, n3;
  sub_module1 U1 (.a(a),  .b(1'b1), .y(n1));
  sub_module2 U2 (.a(n1), .b(1'b0), .y(n2));
  sub_module2 U3 (.a(b),  .b(d),    .y(n3));
  assign y = c | (b & n1);
endmodule
```

**Analysis:** Tracing through the intermediate signals:
- `n1 = a & 1 = a`
- `n2 = a ^ 0 = a` (unused in final output)
- `n3 = b ^ d` (unused in final output)
- `y = c | (a & b)`

So the final optimized output only needs one AND gate and one OR gate.

> **Note:** The `flatten` command must be run before `opt_clean` so that Yosys can optimize across sub-module boundaries.

**Synthesis Commands:**
```bash
yosys
read_liberty -lib /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_module_opt.v
synth -top multiple_module_opt
flatten
opt_clean -purge
abc -liberty /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![multiple_module_opt synthesis result — AND and OR gate](images/image32.png)

We can see that the final optimization shows an AND gate and an OR gate, as we expected.

---

## Sequential Logic Optimizations

---

### Code 6 — D Flip-Flop with Asynchronous Reset (flop retained)

```verilog
module dff_const1(input clk, input reset, output reg q);
  always @(posedge clk, posedge reset)
  begin
    if(reset)
      q <= 1'b0;
    else
      q <= 1'b1;
  end
endmodule
```

**Analysis:** On reset, `q = 0`. On every clock posedge without reset, `q = 1`. Even though `q` always ends up as `1`, the synthesis tool **cannot** remove the flip-flop here because there is still a moment in time when `q` is `0` (right after reset). The flop must be retained.

**Synthesis Commands:**
```bash
yosys
read_liberty -lib /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const1.v
synth -top dff_const1
dfflibmap -liberty /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt_clean -purge
abc -liberty /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![dff_const1 synthesis result — flop retained](images/image33.png)

This is the optimized version of the D flip-flop file. The flip-flop is retained in the synthesized output.

---

### Code 7 — D Flip-Flop (optimized away — constant output)

```verilog
module dff_const2(input clk, input reset, output reg q);
  always @(posedge clk, posedge reset)
  begin
    if(reset)
      q <= 1'b1;
    else
      q <= 1'b1;
  end
endmodule
```

**Analysis:** Both the reset and the normal clock branch assign `q = 1`. Since `q` is **always** `1` regardless of any input, the synthesis tool can optimize the entire flip-flop away and replace it with a constant logic `1` output.

**Synthesis Commands:**
```bash
yosys
read_liberty -lib /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const2.v
synth -top dff_const2
dfflibmap -liberty /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt_clean -purge
abc -liberty /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![dff_const2 synthesis result — constant 1 output](images/image34.png)

We can see that the output is always `1` regardless of input. The flip-flop is completely removed.

---

## Key Takeaways from Day 3

- `opt_clean -purge` sweeps away redundant logic during synthesis
- Synthesis tools are smart enough to reduce complex nested logic to simple gates
- A flip-flop can only be optimized away if its output is **constant at all times**, including during reset
- `flatten` is needed before optimizing multi-module designs

---

