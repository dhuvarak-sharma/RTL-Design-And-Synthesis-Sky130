# Day 4 — Gate Level Simulation (GLS) and Simulation-Synthesis Mismatches

## Theory Topics Covered

### Gate-Level Simulation (GLS)

Gate-Level Simulation is performed after synthesis using the generated gate-level netlist. Unlike RTL simulation, GLS verifies how the synthesized hardware behaves using actual gates from the standard cell library.

GLS helps detect:

- Timing-related issues
- Synthesis mismatches
- Improper RTL coding practices
- Hardware inference problems

---

### Blocking vs Non-Blocking Assignments

Verilog supports two major assignment styles:

- Blocking assignments (`=`)
- Non-blocking assignments (`<=`)

Blocking assignments execute sequentially within an always block, while non-blocking assignments update simultaneously after the block execution.

Proper usage is essential for avoiding unintended hardware behavior.

---

### Synthesis-Simulation Mismatch

A synthesis-simulation mismatch occurs when RTL simulation behavior differs from the behavior of the synthesized hardware.

This commonly happens due to:

- Incorrect sensitivity lists
- Incomplete conditional statements
- Improper assignment styles
- Unintended latch inference

---

### Importance of Proper Sensitivity Lists

Sensitivity lists determine when an always block executes. Incorrect sensitivity lists can produce simulation errors and mismatches.

Using `always @(*)` ensures that the block responds to changes in all relevant signals.

---

---

## Labs

On Day 4, **Gate Level Simulation (GLS)** was introduced — the process of simulating the synthesized netlist instead of the original RTL code. This allows verification that the synthesized design behaves identically to what was intended. Two important classes of bugs that GLS can help expose were also studied.

---

## What is Gate Level Simulation (GLS)?

When Verilog is written, the behavioral RTL code is simulated to check functionality. After synthesis, the result is a **netlist** — a description of the actual gates and connections. GLS means running simulation on this netlist.

**Why bother?** Because sometimes the synthesized netlist doesn't behave the same as the RTL code. This is called a **synthesis-simulation mismatch**, and it can happen due to:
1. Incorrect sensitivity lists in `always` blocks
2. Improper use of blocking vs. non-blocking assignments

GLS uses the same testbench but feeds it the synthesized netlist along with the gate-level models of the Sky130 standard cells.

---

## 1. Invoking and Experimenting with GLS (Ternary Operator MUX)

```verilog
module ternary_operator_mux (input i0, input i1, input sel, output y);
  assign y = sel ? i1 : i0;
endmodule
```

Here, the output `y` becomes `i1` when `sel = 1`, otherwise `y = i0`. This is the functionality of the ternary operator.

![RTL simulation waveform of ternary_operator_mux](../images/image35.png)

After synthesizing the code, the following output is obtained:

![Synthesized schematic of ternary_operator_mux](../images/image36.png)

GLS was then carried out on this file.

### Process of Performing GLS

> **Note:** When running GLS, the Sky130 primitive and cell model files must also be passed in, because iverilog needs to know the detailed behavior of each gate in the netlist.

```bash
iverilog /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/primitives.v \
         /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/sky130_fd_sc_hd.v \
         ternary_operator_mux.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```

![GLS waveform of ternary_operator_mux](../images/image37.png)

The GLS waveform matches the RTL simulation waveform — no mismatch. The design is correct.

---

## 2. Performing GLS on a Bad MUX

```verilog
module bad_mux (input i0, input i1, input sel, output reg y);
  always @ (sel) begin
    if (sel)
      y <= i1;
    else
      y <= i0;
  end
endmodule
```

This code attempts to implement a multiplexer using an `always` block, but because the sensitivity list contains only `sel`, changes in `i0` or `i1` may not update the output correctly during simulation. The code runs like a flop instead of a mux.

In order to correct the mistake, in the `always` block, instead of `sel`, `*` needs to be given so that any changes in any of the signals will give a corresponding change in the output.

![RTL simulation waveform of bad_mux — flop-like behaviour](../images/image38.png)

On viewing the waveform of the original `bad_mux.v`, it can be seen that `y` changes only according to `sel`. In the beginning, there was no activity in `sel`, so `y` was constantly holding the initial value of `i0`. However, when `sel` becomes high, `y` takes the value of `i1`. Then, when `sel` becomes low, `y` takes on the value of `i0`, which is high, and `y` remains high. Thus, `y` changes only with a change in the activity of `sel`, and not according to the incoming signals — the system acts like a flop. This is a **synthesis-simulation mismatch**.

The code was then synthesized:

```bash
yosys
read_liberty -lib /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog bad_mux.v
synth -top bad_mux
abc -liberty /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr bad_mux_net.v
```

The netlist was then simulated after performing GLS:

![GLS waveform of bad_mux — correct mux behaviour after GLS](../images/image39.png)

After performing GLS, it can be seen that the `y` (output) signal now varies with any change in the input signals, and not just due to the `sel` signal. The system now behaves like a multiplexer and not a flop. Thus, the bad mux has been fixed.

---

## 3. Analysing the Disadvantages of Blocking Statements

```verilog
module blocking_caveat (input a, input b, input c, output reg d);
  reg x;
  always @ (*) begin
    d = x & c;
    x = a | b;
  end
endmodule
```

In this code, within the `always` block, in the first statement, `d` takes on a value using an **old value** of `x`, and then in the second statement the value of `x` is updated to the newest value. Thus, `d` always uses the old value of `x`, and ends up being delayed by one cycle. This is not a syntactical issue, but a **logical/semantic error**.

![RTL simulation waveform of blocking_caveat — one-cycle delay](../images/image40.png)

At the selected point it can be seen that `a = 0`, `b = 0`, `c = 1`. Thus `a | b = x = 0`, and `x & c = 0`. However, `d` (the output signal) has the value of `1`. This is because the **previous value** of `x` is used, where `x` was `1`. Thus `x & c` results in `1`. There is a one-cycle delay in the system.

This was then fixed by performing GLS:

```bash
yosys
read_liberty -lib /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog blocking_caveat.v
synth -top blocking_caveat
abc -liberty /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr blocking_caveat_net.v
show
```

![Synthesized schematic of blocking_caveat](../images/image41.png)

The new waveform was then viewed:

![GLS waveform of blocking_caveat — cycle delay fixed](../images/image42.png)

From this waveform, it can be seen that the output `d` is now receiving the **new values** of `a | b` as input. Thus, it is in sync with the input signals `a`, `b`, and `c`. The one-cycle delay has been fixed due to gate-level synthesis.

---

## Key Takeaways from Day 4

- **GLS** = simulating the synthesized netlist using gate-level cell models
- GLS reveals **synthesis-simulation mismatches** that RTL simulation alone would miss
- Always use `*` in the sensitivity list of combinational `always` blocks
- Be careful with the **order of blocking assignments** — they execute sequentially, not simultaneously
- Non-blocking assignments (`<=`) are safer for sequential logic; blocking (`=`) should be used carefully in combinational blocks

---
