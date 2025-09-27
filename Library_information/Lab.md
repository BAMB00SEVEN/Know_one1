


### File 2: `lab.md`

````markdown
# Day 2: Lab Workflow - Simulation and Synthesis

This guide provides the step-by-step commands for simulating a flip-flop design using Icarus Verilog and synthesizing it into a gate-level netlist with Yosys.

---

## Part 1: RTL Implementation and Simulation

This section covers the creation of a Verilog module for a D-Flip-Flop with an asynchronous reset and its functional verification.

### ► Step 1: Create the RTL File

Create a file named `dff_asyncres.v` and add the following Verilog code. This design describes a D-type flip-flop that is triggered by the positive edge of the clock and reset by the positive edge of an asynchronous reset signal.

```verilog
// dff_asyncres.v
module dff_asyncres (
    input clk, 
    input async_reset, 
    input d, 
    output reg q
);
  always @ (posedge clk, posedge async_reset)
    if (async_reset)
      q <= 1'b0;
    else
      q <= d;
endmodule
````

> **Note:** You will also need a corresponding testbench file (e.g., `tb_dff_asyncres.v`) to drive the inputs (`clk`, `async_reset`, `d`) and observe the output `q`.

### ► Step 2: Compile and Simulate with Icarus Verilog

Execute the following commands in your terminal to compile the design and its testbench, run the simulation, and generate a waveform file.

1.  **Compile the Verilog files:**
    The `iverilog` command invokes the compiler, which parses your RTL and testbench files and creates an executable simulation file, `a.out` by default.

    ```bash
    iverilog dff_asyncres.v tb_dff_asyncres.v
    ```

2.  **Run the simulation:**
    Executing the compiled output will run the simulation defined in your testbench. This process generates a Value Change Dump (`.vcd`) file, which logs the signal activity.

    ```bash
    ./a.out
    ```

3.  **View the waveforms:**
    Use GTKWave to open the `.vcd` file and visualize the waveforms. This allows you to graphically verify that the flip-flop behaves as expected under different input conditions.

    ```bash
    gtkwave tb_dff_asyncres.vcd
    ```

-----

## Part 2: Synthesis with Yosys

This section walks through the process of converting the RTL design into a gate-level netlist using standard cells from the SKY130 PDK.

### ► Step 1: Launch Yosys

Start the Yosys synthesis suite in interactive mode.

```bash
yosys
```

### ► Step 2: Read the Timing Library

This command parses the `.lib` file, loading the standard cell definitions (logic gates, flip-flops) and their timing characteristics into the tool's memory. Yosys needs this information to map the generic logic to physical cells.

```bash
read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### ► Step 3: Read the Verilog Design

Load your RTL code into Yosys. The tool will convert your Verilog code into its own internal representation.

```bash
read_verilog dff_asyncres.v
```

### ► Step 4: Synthesize the Design

The `synth` command is a generic, high-level command that runs a default synthesis script. It performs coarse-grain synthesis, converting behavioral constructs (like `if` statements) into a generic gate-level representation (e.g., AND, OR, MUX, DFF). The `-top` argument specifies the top-level module of the design.

```bash
synth -top dff_asyncres
```

### ► Step 5: Map Flip-Flops to Library Cells

The `dfflibmap` command specifically targets the generic DFF cells created by the `synth` command and replaces them with specific flip-flop standard cells available in the provided `.lib` file.

```bash
dfflibmap -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### ► Step 6: Perform Technology Mapping

The `abc` command is a powerful logic synthesis and verification tool integrated into Yosys. It performs fine-grained technology mapping, taking the generic logic gates and optimizing them into the specific standard cells from the loaded library (`-liberty`), aiming to meet timing and area constraints.

```bash
abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### ► Step 7: Visualize the Synthesized Netlist

The `show` command generates a visual representation of the current gate-level netlist. This is extremely useful for verifying that the synthesis process produced the expected logic structure. For our DFF, you should see it mapped to a SKY130 library cell like `sky130_fd_sc_hd__dfxtp_1`.

```bash
show
```

```
```
