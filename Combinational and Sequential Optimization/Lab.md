

### File 2: `lab.md`

````markdown
# Day 3 Lab: Observing Synthesis Optimizations

In this lab, we will write several small Verilog modules and observe how the Yosys synthesis tool automatically optimizes them. The goal is to see the theoretical concepts from today's session in action.

---

## Synthesis Workflow

For each lab, you will perform the following synthesis steps. The key difference from previous days is the addition of the `opt_clean -purge` command, which aggressively removes unused logic.

1.  Launch Yosys:
    ```bash
    yosys
    ```
2.  Load the standard cell library:
    ```bash
    read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
    ```
3.  Read in your Verilog design file (e.g., `lab1.v`):
    ```bash
    read_verilog <your_verilog_file.v>
    ```
4.  Run the standard synthesis script:
    ```bash
    synth -top <your_top_module_name>
    ```
5.  **Perform aggressive optimization and cleanup**:
    ```bash
    opt_clean -purge
    ```
6.  Map the design to the technology library:
    ```bash
    abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
    ```
7.  Visualize the final, optimized netlist:
    ```bash
    show
    ```

---

## Lab Experiments

### ► Lab 1 & 2: Mux Simplification

This logic describes a simple 2-to-1 multiplexer.
* **Lab 1 Code**: `assign y = a ? b : 1'b0;`
* **Lab 2 Code**: `assign y = a ? 1'b1 : b;`

```verilog
// Lab 1 Verilog (`lab1.v`)
module opt_check1 (input a, input b, output y);
	assign y = a ? b : 1'b0; // If a is 1, y=b. Else, y=0.
endmodule

// Lab 2 Verilog (`lab2.v`)
module opt_check2 (input a, input b, output y);
	assign y = a ? 1'b1 : b; // If a is 1, y=1. Else, y=b.
endmodule
````

> **Observation**: After synthesis, these should map to simple AND and OR gate implementations, respectively.

### ► Lab 3 & 4: Redundant Logic Removal

The logic in these modules seems complex but contains redundancies that the optimizer can easily remove.

  * **Lab 3 Code**: This is identical to Lab 2 and shows that synthesis will produce the same result.
  * **Lab 4 Code**: `assign y = a ? (b ? (a & c) : c) : (!c);`

<!-- end list -->

```verilog
// Lab 4 Verilog (`lab4.v`)
module opt_check4 (input a, input b, input c, output y);
    // This complex statement simplifies to y = (a & c) | (~a & ~c)
    // which is the logic for an XNOR gate.
    assign y = a ? (b ? (a & c) : c) : (!c);
endmodule
```

> **Observation**: The complex nested ternary in Lab 4 will be simplified by constant propagation. When `a` is `1`, the expression `(b ? (1 & c) : c)` simplifies to just `c`. When `a` is `0`, the expression becomes `!c`. The final logic simplifies to `y = a ? c : !c;` which is an XNOR gate.

### ► Lab 5 & 6: Constant Flip-Flop Optimization

These labs demonstrate how optimizers handle sequential circuits with constant drivers.

  * **Lab 5 Code**: A DFF that is always loaded with `1`.
  * **Lab 6 Code**: A DFF whose output is always `1`, even during reset.

<!-- end list -->

```verilog
// Lab 5 Verilog (`lab5.v`)
module dff_const1(input clk, input reset, output reg q);
    always @(posedge clk, posedge reset) begin
        if(reset)
            q <= 1'b0;
        else
            q <= 1'b1; // Data input is tied high
    end
endmodule

// Lab 6 Verilog (`lab6.v`)
module dff_const2(input clk, input reset, output reg q);
    always @(posedge clk, posedge reset) begin
        if(reset)
            q <= 1'b1;
        else
            q <= 1'b1; // Output is always high
    end
endmodule
```

> **Observation**: In Lab 6, the `q` output is always `1`. The optimizer will recognize this, remove the flip-flop entirely, and tie the output `q` directly to the `Vdd` (logic 1) supply. The clock and reset inputs will be left unused.

```
```
