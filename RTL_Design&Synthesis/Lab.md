###  MODULE --- 1
------
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

** Make sure to change your directory into `verilog_files` before running the upcoming commands, as this folder contains all the verilog files for the modules to be executed
>
>
><img width="1622" height="1103" alt="Image" src="https://github.com/user-attachments/assets/8ecc81d1-518f-4c4c-9e5c-5a769344ca8b" />

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
    <img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/ec780bdc-2228-40dd-adf1-cfd957c7c71b" />

3.  **View**: This opens the waveform viewer to visually inspect signals.
    ```bash
    gtkwave dump.vcd
    ```
<img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/92c413ff-246f-4516-b670-3019be41a78a" />

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
    <img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/a4705bbe-8ddb-4a1a-819f-198d05e77332" />

3.  **Read Design**: Loads your Verilog file.
    ```yosys
    read_verilog good_latch.v
    ```
4.  **Synthesize**: Converts the behavioral Verilog into a generic gate-level design.
    ```yosys
    synth -top good_latch
    ```
    <img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/a9e95808-e01e-4aa7-b8d4-a917122d3cb6" />

5.  **Map to Technology**: Maps the generic gates to the specific, physical cells from the loaded library.
    ```yosys
    abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    ```
    <img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/ad787432-343d-4852-9717-bd4bf0965375" />

6.  **Visualize Netlist**: Displays a graphical schematic of the synthesized circuit.
    ```yosys
    show
    ```
    <img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/684ab650-6025-4727-accc-9c21bea34f4e" />

7.  **Save Netlist**: Saves the final, technology-mapped netlist to a file.
    ```yosys
    write_verilog my_latch.v
    ```
synth -top good_latch
This is the core synthesis command that performs a generic, technology-independent synthesis. It transforms the behavioral Verilog constructs (like always blocks and if statements) into a netlist of generic logic gates (like AND, OR, MUX, DFF). The -top good_latch flag tells Yosys which module is the top-level entity of the design to synthesize.

abc -liberty ...
This command performs technology mapping. The previous synth command created a circuit using generic gates. The abc tool takes this generic netlist and maps it to the specific, physical standard cells defined in the Liberty (.lib) file you loaded. It's a crucial optimization step that reworks the circuit to use the actual cells available in your target technology (e.g., sky130), optimizing for criteria like area and delay.

show
This command is a visualization and debugging tool. It generates a graphical viewer that displays the current state of the design as a schematic. It's used to visually inspect the netlist, allowing you to see exactly how the logic gates from the library have been interconnected to implement your design. It's an excellent way to confirm that the synthesis process produced a logical structure.


-----

### 4\. Final Verification

Finally, inspect the generated netlist file to understand the output of the synthesis process.

Open in a Text Editor: Use any command-line editor to view the contents of the output file. The ! prefix in Yosys allows you to run shell commands.

```yosys
!gedit my_latch.v
```
 <img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/f193d732-56dc-4f8c-a0b9-710fea5bfc55" />

Inside this file, you will no longer see behavioral code but a structural description—a list of standard cell instances from the library and the wires that connect them.
