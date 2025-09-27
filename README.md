# Verilog for Synthesis: Concepts and Labs

This repository is a collection of notes and hands-on labs exploring key concepts in digital logic design using Verilog. The focus is on writing clean, synthesizable code and understanding the journey from Register-Transfer Level (RTL) to an optimized gate-level netlist.

Made by ::::: `#ff0000`Parth_Batra

---
## Key Topics Covered

### Writing Synthesizable Verilog
* **Blocking (`=`) vs. Non-Blocking (`<=`) Assignments**: The fundamental rules for correctly modeling combinational and sequential logic.
* **Preventing Inferred Latches**: Understanding how incomplete `if` or `case` statements create unintentional memory and learning best practices to avoid them.
* **Flip-Flop & MUX Coding Styles**: Implementing common digital building blocks in a clean, efficient manner.

### The Synthesis and Optimization Flow
* **From RTL to Netlist**: The process of converting abstract Verilog into a physical gate-level netlist using tools like Yosys.
* **The Role of Timing Libraries (`.lib`)**: How synthesis tools use standard cell libraries to understand gate characteristics under different PVT (Process, Voltage, Temperature) corners.
* **Automatic Optimization**: Observing how synthesis tools simplify logic through constant propagation, redundant logic removal, and purging of unused hardware.

### Verification and Validation
* **Gate-Level Simulation (GLS)**: The critical step of simulating the post-synthesis netlist to confirm that the synthesized hardware matches the original design intent.
* **Simulation-Synthesis Mismatch**: Identifying common RTL coding pitfalls that cause discrepancies between the initial simulation and the final hardware behavior.

### Scalable Hardware Design
* **Procedural `for` Loops**: A compact method for describing large, repetitive combinational logic structures like multiplexers and encoders.
* **Structural `generate` Blocks**: A powerful feature for creating multiple instances of modules or gates, ideal for building regular structures like Ripple-Carry Adders.

---
## Repository Structure

The content is organized into folders, each containing:

* **`theory.md`**: Explanations of core digital design concepts.
* **`lab.md`**: Hands-on exercises with Verilog code to demonstrate and reinforce the theory.

---
## Tools Used

* **Yosys**: For logic synthesis.
* **Icarus Verilog**: For Verilog simulation.
* **GTKWave**: For viewing simulation waveforms.
* **SKY130 PDK**: As the target standard cell library.
