
# Lab: Good vs. Bad RTL and Gate-Level Simulation

This lab will demonstrate the correct flow for Gate-Level Simulation (GLS) and highlight common RTL coding mistakes that lead to simulation-synthesis mismatches.

---

## Part 1: Correct GLS Flow (Lab 1)

Here, we will see a simple multiplexer, synthesize it, and run a gate-level simulation to verify it.

### ► Step 1: Choose the RTL

Choose a Verilog file for a simple 2:1 MUX using a synthesizable continuous assignment.

><img width="2908" height="1271" alt="Image" src="https://github.com/user-attachments/assets/9fb67f8c-e1c9-4b08-9e6e-c50b68489580" />

### ► Step 2: Synthesize with Yosys

Run the MUX through the standard Yosys synthesis flow to generate a gate-level netlist (e.g., `my_ter_mux_gls.v`).

**Note : you can give any name to the verilog file while writing it as i use "my" prefix for every netlist, here i used `my_ter_mux_gls.v`**

```bash
read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog my_ter_mux_gls.v
```

### ► Step 3: Run Gate-Level Simulation

Use Icarus Verilog to compile the necessary files. The command includes the **gate definitions** (`primitives.v` and `sky130_fd_sc_hd.v`), your **synthesized netlist**, and your **testbench**.

```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v my_ter_mux_gls.v tb_ternary_operator_mux.v

```
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/5abac324-01d1-4cb0-b3e0-92c0da1068fd" />
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/0dd4dd2d-995d-4626-b83f-f8ecf2890876" />

> **Note**: The simulation should pass, as the original RTL was clean and synthesizable. Also make sure to enter the correct path for simulation

-----

## Part 2: Common RTL Pitfalls (Lab 2)

These labs showcase incorrect coding styles and their consequences.

### ► Pitfall 1: Bad Combinational Logic 

This MUX is coded using an `always` block, but with two critical errors.

```verilog
// bad_mux.v
module bad_mux (input i0, input i1, input sel, output reg y);
  // INCORRECT CODE
  always @ (sel) begin // <-- ERROR 1: Incomplete sensitivity list
    if (sel)
      y <= i1;         // <-- ERROR 2: Non-blocking in combo logic
    else 
      y <= i0;
  end
endmodule
```
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/949acc33-ccee-4960-93a1-6a04d1c61aec" />

**The Fix:**

1.  The sensitivity list must include **all** signals read inside the block (`i0`, `i1`, `sel`). Use `*` to do this automatically.
2.  Combinational logic must use **blocking** assignments (`=`) to avoid inferring latches.

<!-- end list -->

```verilog
// CORRECTED MUX
always @ (*) begin
  if (sel)
    y = i1;
  else
    y = i0;
end
```
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/8d2596a0-ec10-4a0e-86b8-9ed4b3f7a0bc" />

### ► Pitfall 2: Blocking Assignment Order (Lab 3)

The order of execution matters for blocking assignments.

```verilog
// blocking_caveat.v
module blocking_caveat (input a, input b, input c, output reg d);
  reg x;
  // INCORRECT CODE
  always @ (*) begin
    d = x & c; // 'd' gets the OLD value of 'x' from the previous run
    x = a | b; // 'x' is updated *after* 'd' has already been calculated
  end
endmodule
```
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/e4b0d47e-9a52-4e22-a6f3-e8cc24889d6d" />


**The Fix:**
Ensure that intermediate signals are calculated *before* they are used within the same block.

```verilog
// CORRECTED ORDER
always @ (*) begin
  x = a | b; // Calculate the new 'x' first
  d = x & c; // Then use the new 'x' to calculate 'd'
end
```
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/140dc6ea-85fc-4524-a535-9a033d34f30d" />


**Key Takeaway**: These "bad" examples will often simulate one way in RTL and synthesize into something completely different, leading to a failed Gate-Level Simulation. Always follow the golden rules of assignment.

```
```
