# Pre-Requisites
---

Before beginning RTL design and synthesis, it is important to understand some basic digital design concepts.

### Combinational vs Sequential Logic
- **Combinational Logic**: Output depends only on present inputs and does not store data. Examples include adders and multiplexers.
- **Sequential Logic**: Output depends on current inputs and previous states using memory elements like flip-flops, synchronized by a clock signal.
### Setup and Hold Times

Reliable hardware operation requires signals to satisfy timing constraints:

- **Setup Time**: Data must remain stable before the clock edge.
- **Hold Time**: Data must remain stable after the clock edge.

Violating these constraints may cause unpredictable behavior.

## Verilog Modeling Styles
- **Structural Modeling**: Describes circuits using explicit gate-level connections.
- **Behavioral Modeling**: Describes circuit functionality using high-level constructs such as if-else statements, allowing synthesis tools to generate the hardware implementation.
