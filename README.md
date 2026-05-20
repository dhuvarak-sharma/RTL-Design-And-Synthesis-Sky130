# RTL Design and Synthesis using Sky130 Technology

> **Author:** Dhuvarak Sriram Sharma  
> **Workshop Reference:** [VSD Sky130 RTL Design and Synthesis Workshop](https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop)

---

## About This Repository

This repository contains my personal lab notes and documentation from the **VSD Sky130 RTL Design and Synthesis Workshop**. Over the course of this workshop, I learned how to simulate, synthesize, and optimize digital logic designs using industry-standard open-source tools — all based on the **Sky130 process node** by SkyWater Technology.

This was my first time working with RTL design tools, and I documented each day's work carefully so that anyone (including future me!) can follow along and reproduce the results.

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **iverilog** | Verilog simulation |
| **GTKWave** | Waveform viewer |
| **Yosys** | RTL synthesis |
| **gVim** | Viewing Verilog source files |
| **Sky130 PDK** | Standard cell library for synthesis |

---

## Repository Structure

```
RTL-Design-And-Synthesis-Sky130/
│
├── README.md          ← You are here (overview of the whole workshop)
├── Day1.md            ← Installation, simulation with iverilog & GTKWave, intro to Yosys
├── Day2.md            ← Liberty files, hierarchical vs flat synthesis, flip-flop types
├── Day3.md            ← Logic optimization (combinational and sequential)
├── Day4.md            ← Gate Level Simulation (GLS), simulation-synthesis mismatches
└── Day5.md            ← If/case statement hazards, for loops, generate blocks
```

---

## Day-by-Day Summary

| Day | Topics Covered |
|-----|----------------|
| [Day 1](./Day1.md) | Cloning the workshop repo, installing tools, simulating a MUX with iverilog and GTKWave, intro to Yosys synthesis |
| [Day 2](./Day2.md) | Understanding the Sky130 `.lib` file, hierarchical vs. flat synthesis, flip-flop types and their synthesis |
| [Day 3](./Day3.md) | Combinational and sequential logic optimizations using Yosys |
| [Day 4](./Day4.md) | Gate Level Simulation (GLS), fixing a bad MUX, understanding blocking statement issues |
| [Day 5](./Day5.md) | Incomplete if/case hazards (latch inference), for loops, generate blocks, ripple carry adder |

---

## Key Concepts Learned

- How to simulate Verilog designs and read waveforms
- How synthesis tools convert RTL code into logic gates using a standard cell library
- The difference between **hierarchical** and **flat** synthesis
- How **asynchronous** and **synchronous** resets/sets work in flip-flops
- What **Gate Level Simulation (GLS)** is and why it matters
- Why incomplete `if`/`case` statements cause **latch inference**
- How to use **for loops** and **generate blocks** to write scalable hardware code

---

## How to Use This Repository

If you'd like to follow along, start by cloning the workshop's Verilog files:

```bash
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
```

Then install the required tools:

```bash
sudo apt install iverilog
sudo apt install gtkwave
```

Follow the notes in each Day's file for step-by-step instructions.

---

## Acknowledgements

- **Kunal Ghosh** and the **VSD team** for designing and providing this workshop
- **SkyWater Technology** for the open-source Sky130 PDK
