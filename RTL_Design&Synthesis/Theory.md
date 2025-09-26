

-----

# RTL Design & Synthesis Flow

This document outlines the fundamental steps in a typical digital design flow, from writing Register Transfer Level (RTL) code to synthesizing it into a gate-level netlist.

## 1\. RTL Design

**RTL Design** is the starting point. It's a way of describing a digital circuit's operation by focusing on how data moves between registers and the logical operations performed on that data.

  - It is a **behavioral representation** of the required hardware specification.
  - This is typically written in a Hardware Description Language (HDL) like **Verilog** or VHDL.

<!-- end list -->

```verilog
// A behavioral description in Verilog
module sample_code (
  input clk,
  input rst,
  output result,
  output done
);

  always @ (posedge clk, posedge rst) begin
    if(rst) begin
      // reset logic
    end
    else begin
      // design logic
    end
  end

endmodule
```

-----

## 2\. Simulation (Verification)

Before converting our abstract design into physical gates, we must ensure it works correctly. This crucial step is called **verification** or **simulation**. Its goal is to check if the RTL design adheres to the specifications.

### The Testbench

To test the design, we need a separate piece of code called a **Testbench**.

  - A testbench is a setup used to apply **stimulus** (also called test vectors) to the design.
  - It acts as a controlled environment that mimics how the design would be used, driving its inputs and checking its outputs. A typical testbench consists of a **Stimulus Generator**, the **Design** under test, and a **Stimulus Observer**.

### The Simulator

A **simulator** is the software tool that executes the design and testbench code to check functionality.

  - It works by monitoring the **input signals**.
  - When an input value changes, the simulator evaluates the logic and updates the **output signals**.
  - The open-source tool used in this flow is **`iverilog`**.

### Simulation Flow

The verification process follows these steps:

1.  **Inputs**: The `iverilog` simulator takes both the **Design** (your RTL code) and the **Testbench** as input.
2.  **Execution**: The simulator runs the testbench, which in turn drives the design's inputs.
3.  **Output (`.vcd`)**: The tool generates a **Value Change Dump (`.vcd`) file**. This is a log file that records every time a signal's value changes during the simulation.
4.  **Visualization**: A waveform viewer like **`gtkwave`** is used to open the `.vcd` file and display the signals graphically over time. This allows you to easily debug and confirm if the design is behaving as expected.

-----

## 3\. Synthesis

**Synthesis** is the process of translating the abstract RTL code into a concrete, physical implementation using logic gates.

  - It's an **RTL to Gate-level translation**.
  - The synthesis tool converts the behavioral code (e.g., `if-else`, `+`, `? :`) into a structural description of standard cells (gates) and the wires connecting them.
  - The output is a **Netlist**, which is essentially a map of which gates to use and how to connect them.

### The Liberty File (`.lib`)

To perform synthesis, the tool needs a catalogue of available logic gates. This is provided by a **`.lib` (Liberty) file**.

  - It is a **collection of logical modules** (standard cells) like AND, OR, NOT, flip-flops, etc., for a specific technology (e.g., SKY130).
  - Crucially, it contains different "flavors" (e.g., slow, medium, fast) of the same gate with different performance characteristics.

### Cell Flavors: The Need for Speed (and Sloth\!) 

Why does a library contain multiple versions of the same gate? It's all about balancing performance, power, and area by carefully managing timing.

#### The Need for Fast Cells (Meeting Setup Time)

To ensure a circuit can run at the desired clock speed, data must travel from one flip-flop to the next and arrive *before* the next clock edge. This is called **setup time**.

  - **Equation**: $T_{CLK} > T_{C \to Q} + T_{COMBI} + T_{SETUP}$
  - To satisfy this, the combinational logic delay ($T_{COMBI}$) must be small. We use **fast cells** to reduce this delay and meet performance targets.

#### The Need for Slow Cells (Fixing Hold Time)

Data from one flip-flop must *not* arrive at the next one *too quickly*, as the new data could corrupt the old data before the flip-flop has a chance to properly capture it. This is called **hold time**.

  - **Equation**: $T_{HOLD} < T_{C \to Q} + T_{COMBI}$
  - If the path between flip-flops is too fast, a hold violation occurs. To fix this, we sometimes need to insert **slow cells** to intentionally add delay to the path.

#### The Physical Trade-Off

Cell speed isn't free. The difference comes down to the size of the transistors inside.

  - **Faster Cells**: Use **wider transistors**. They can source more current to charge/discharge the circuit's capacitance quickly, leading to **low delay**. The penalty is **more area and higher power consumption**.
  - **Slower Cells**: Use **narrow transistors**. They have **higher delay** but are more efficient, consuming **less area and less power**.

### Synthesis Constraints

We can't just use fast cells everywhere (due to power/area) or slow cells everywhere (due to performance). We need to guide the synthesis tool.

  - This guidance is given in the form of **Constraints** (e.g., target clock frequency, power budget).
  - An open-source synthesis tool like **Yosys** then performs a complex optimization, selecting the optimal "flavor" of each cell to meet the timing requirements (**setup and hold**) while minimizing area and power.
