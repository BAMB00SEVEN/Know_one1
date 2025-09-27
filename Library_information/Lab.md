
### ► Part 1: Compile and Simulate with Icarus Verilog

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

><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/d5f8bc05-b0f9-43d9-a740-d0fa75984c6f" />

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
