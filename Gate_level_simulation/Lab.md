
### File 2: `lab.md`


# Day 4 Lab: Good vs. Bad RTL and Gate-Level Simulation

This lab will demonstrate the correct flow for Gate-Level Simulation (GLS) and highlight common RTL coding mistakes that lead to simulation-synthesis mismatches.

---

## Part 1: Correct GLS Flow (Labs 1-3)

Here, we will create a simple multiplexer, synthesize it, and run a gate-level simulation to verify it.

### ► Step 1: Create the RTL

Create a Verilog file for a simple 2:1 MUX using a synthesizable continuous assignment.

```verilog
// ternary_operator_mux.v
module ternary_operator_mux (
    input i0, 
    input i1, 
    input sel, 
    output y
);
  assign y = sel ? i1 : i0;
endmodule
````

### ► Step 2: Synthesize with Yosys

Run the MUX through the standard Yosys synthesis flow to generate a gate-level netlist (e.g., `mux_netlist.v`).

```bash
# Yosys script/commands
read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog mux_netlist.v
```

### ► Step 3: Run Gate-Level Simulation

Use Icarus Verilog to compile the necessary files. The command includes the **gate definitions** (`primitives.v` and `sky130_fd_sc_hd.v`), your **synthesized netlist**, and your **testbench**.

```bash
iverilog -o gls_sim.out /path/to/primitives.v /path/to/sky130_fd_sc_hd.v mux_netlist.v your_testbench.v
```

> **Note**: The simulation should pass, as the original RTL was clean and synthesizable.

-----

## Part 2: Common RTL Pitfalls (Labs 4-7)

These labs showcase incorrect coding styles and their consequences.

### ► Pitfall 1: Bad Combinational Logic (Lab 4 & 5)

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

### ► Pitfall 2: Blocking Assignment Order (Lab 6 & 7)

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

**The Fix:**
Ensure that intermediate signals are calculated *before* they are used within the same block.

```verilog
// CORRECTED ORDER
always @ (*) begin
  x = a | b; // Calculate the new 'x' first
  d = x & c; // Then use the new 'x' to calculate 'd'
end
```

> **Key Takeaway**: These "bad" examples will often simulate one way in RTL and synthesize into something completely different, leading to a failed Gate-Level Simulation. Always follow the golden rules of assignment.

```
```
