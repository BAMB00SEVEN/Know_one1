Got it. Here's the concise guide with each command in its own copy-able snippet.

## RTL Workflow: Simulation & Synthesis

### Objective

To simulate a Verilog module for functional correctness and then synthesize it into a gate-level netlist.

-----

### 1\. Setup

Prepare the workspace by creating a directory and cloning the required files.

```bash
mkdir -p vsd/VLSI && cd vsd/VLSI
```

```bash
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop
```

-----

### 2\. Simulation (with iverilog)

This phase verifies that the RTL code's logic is correct.

1.  **Compile**: This compiles the design and its testbench into an `a.out` file.
    ```bash
    iverilog good_latch.v tb_good_latch.v
    ```
2.  **Run**: This executes the simulation and creates a `dump.vcd` waveform file.
    ```bash
    ./a.out
    ```
3.  **View**: This opens the waveform viewer to visually inspect signals.
    ```bash
    gtkwave dump.vcd
    ```

-----

### 3\. Synthesis (with Yosys)

This phase converts the verified RTL into a netlist of standard cells.

1.  **Launch Yosys**:
    ```bash
    yosys
    ```
2.  **Load Library**: Loads the available standard cells for the target technology.
    ```yosys
    read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    ```
3.  **Read Design**: Loads your Verilog file.
    ```yosys
    read_verilog good_latch.v
    ```
4.  **Synthesize**: Converts the behavioral Verilog into a generic gate-level design.
    ```yosys
    synth -top good_latch
    ```
5.  **Map to Technology**: Maps the generic gates to the specific, physical cells from the loaded library.
    ```yosys
    abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    ```
6.  **Visualize Netlist**: Displays a graphical schematic of the synthesized circuit.
    ```yosys
    show
    ```
7.  **Save Netlist**: Saves the final, technology-mapped netlist to a file.
    ```yosys
    write_verilog my_latch.v
    ```

-----

### 4\. Final Verification

Inspect the output file (`my_latch.v`), which now contains a structural description of interconnected library cells.

```yosys
!gedit my_latch.v
```
 

