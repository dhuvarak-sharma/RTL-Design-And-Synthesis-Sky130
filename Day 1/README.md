# Day 1 — Installation and Reading a Waveform

[← Back to Main README](./README.md)
---
---

### Introduction to Icarus Verilog

Icarus Verilog (iverilog) is an open-source Verilog simulator used for compiling and simulating RTL designs. During the workshop, it was used to:

- Compile Verilog RTL files
- Execute simulations
- Generate waveform dump files (.vcd)
- Verify hardware behavior before synthesis

---

### Introduction to GTKWave

GTKWave is a waveform visualization tool used for viewing signal transitions generated during simulation. It helps engineers analyze timing relationships, debug logic issues, and verify the correctness of digital designs.

Waveforms generated from Value Change Dump (VCD) files were inspected throughout the workshop to study signal behavior.

---

### Introduction to Yosys

Yosys is an open-source RTL synthesis suite used to convert Verilog RTL code into gate-level netlists using standard cell libraries. It plays a major role in digital front-end VLSI design.

During the workshop, Yosys was used for:

- RTL synthesis
- Logic optimization
- Technology mapping
- Netlist generation
- Hardware visualization

---

### Standard Cell Libraries

Standard cell libraries contain pre-designed hardware cells such as AND gates, OR gates, multiplexers, flip-flops, buffers, and inverters. These cells are used by synthesis tools to map RTL logic into real hardware.

The SKY130 standard cell library was used throughout the workshop.

---

### Gate Flavours (Area, Power, Performance)

Different versions of the same logic gate may exist in a library with different design tradeoffs.

- High-performance cells prioritize speed
- Low-power cells reduce power consumption
- Small-area cells reduce silicon usage

These different implementations are commonly referred to as gate flavours.

---
---

## Labs

On Day 1, I set up the entire lab environment from scratch. This involved cloning the workshop repository, installing the simulation tools (iverilog and GTKWave), running my first simulation, and getting a first look at how Yosys synthesizes a design. I used a simple **2:1 Multiplexer (MUX)** as the example design throughout the day.

---

## 1. Cloning the Workshop Repository

The first step was to download all the Verilog source files and library files needed for the workshop:

```bash
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
```

![Cloning the repository](images/image1.png)

---

## 2. Installing the Required Tools

```bash
sudo apt install iverilog
sudo apt install gtkwave
```

- **iverilog** — an open-source Verilog simulator. It compiles our Verilog code and runs the simulation.
- **GTKWave** — a waveform viewer. It reads the simulation output and lets us see how signals change over time, like an oscilloscope on a computer.

![Installing iverilog](images/image2.png)

![Installing gtkwave](images/image3.png)

The system is now ready, and we can perform future labs in the system.

---

## 3. Working with iverilog and GTKWave

We will be loading the files present in the `verilog_files` folder onto iverilog and then viewing the waveform on GTKWave.

```bash
iverilog good_mux.v tb_good_mux.v
```

![Running iverilog on good_mux](images/image4.png)

When we load the file onto the simulator iverilog, a file called `a.out` is created. On running the `a.out` file, a VCD (Value Change Dump) file is generated. Loading this VCD file onto GTKWave lets us view the waveform of the `good_mux.v` file.

```bash
./a.out
gtkwave tb_good_mux.vcd
```

![GTKWave waveform of good_mux](images/image5.png)

![GTKWave waveform zoomed](images/image6.png)

---

## 4. Viewing the File Structure of a Good MUX

To open and compare the testbench and the design side-by-side in gVim:

```bash
gvim tb_good_mux.v -o good_mux.v
```

![gVim file structure view](images/image7.png)

The **design file** (`good_mux.v`) contains the actual hardware logic. The **testbench** (`tb_good_mux.v`) is a wrapper that feeds different inputs to the design so we can check if it behaves correctly.

---

## 5. Introduction to Yosys (Synthesis)

**Yosys** is an open-source synthesis tool. Its job is to take our RTL (Register Transfer Level) Verilog code and convert it into an actual gate-level netlist — meaning it figures out which real hardware cells (AND gates, OR gates, flip-flops, etc.) from the Sky130 library should be used.

### Step 1 — Invoke Yosys

```bash
yosys
```

### Step 2 — Read the Standard Cell Library

```bash
read_liberty -lib /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This tells Yosys which real hardware cells it is allowed to use when synthesizing our design.

### Step 3 — Read the Design and Synthesize

```bash
read_verilog good_mux.v
synth -top good_mux
```

> **Note:** If the design contains flip-flops, a special `dfflibmap` command must also be run. Since `good_mux.v` is a purely combinational circuit (no memory elements), we skip that step here.

### Step 4 — Map to Gates and View

```bash
abc -liberty /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The `abc` command is what actually converts the RTL logic into real gates from the library. `show` opens a visual diagram of the synthesized circuit.

![Yosys synthesized schematic of good_mux](images/image8.png)

---

## 6. Writing the Netlist

Finally, we write out the synthesized gate-level netlist to a new Verilog file:

```bash
write_verilog good_mux_netlist.v
!gvim good_mux_netlist.v
```

![good_mux netlist in gVim](images/image9.png)

The netlist is a Verilog file that no longer contains behavioral code — instead it describes the design purely in terms of the actual gates used from the Sky130 library.

---

## Key Takeaways from Day 1

- The simulation flow is: `iverilog` → `./a.out` → `gtkwave`
- A **VCD file** is the bridge between simulation and waveform viewing
- Yosys synthesis flow: `read_liberty` → `read_verilog` → `synth` → `abc` → `write_verilog`
- The `abc` command is what maps our design to real hardware gates

---

[Next: Day 2 →](./Day2.md)
