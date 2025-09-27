

# Observing Synthesis Optimizations

In this lab, we will write several small Verilog modules and observe how the Yosys synthesis tool automatically optimizes them. The goal is to see how synthesis tools perform logic reduction and remove redundant or unused hardware based on the RTL code.

-----

## Synthesis Workflow

For each lab, you will perform the following synthesis steps. The key difference from previous labs as we are using the optimized moduless, is the addition of the `opt_clean -purge` command, which aggressively removes unused logic.

**use it between `synth -top` command and ` abc -liberty` command** 

> For finding the names of modules use these commands as shown in the image
> <img width="2908" height="1271" alt="Image" src="https://github.com/user-attachments/assets/2117db76-b51a-4553-a379-2b828d65db07" />

-----

## Lab Experiments

### ► Lab 1: Redundant Mux (`opt_3`)

This module describes a 2-to-1 multiplexer where both data inputs are identical. This makes the selection logic completely redundant.

```verilog
// opt_3.v
module opt_3 (
    input sel, 
    input a, 
    output y
);
  // Both paths of the mux select the same input 'a'
  assign y = sel ? a : a;
endmodule
```
> <img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/175c3b9a-6576-4814-8b1b-5fe3bcb32b26" />


> **Observation**: The synthesis tool will immediately identify that the output `y` is always equal to `a`, regardless of the value of `sel`. The entire multiplexer logic will be optimized away, and the final netlist will simply be a direct wire connecting input `a` to output `y`. The `sel` input will be left unconnected and purged.
---


### ► Lab 2: Constant-Input Flip-Flop (`dff_const1`)

This lab demonstrates how synthesis handles a sequential circuit where the data input to a flip-flop is a constant value.

```verilog
// dff_const1.v
module dff_const1(
    input clk, 
    input reset, 
    output reg q
);
  always @(posedge clk, posedge reset) begin
    if(reset)
      q <= 1'b0;
    else
      q <= 1'b1; // Data input is always tied to logic '1'
  end
endmodule
```
>
<img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/6b27188a-4529-4148-a350-3074d29b6dc1" />

> **Observation**: The tool will synthesize a real flip-flop. However, because the data being loaded is always `1'b1` (when not in reset), the optimizer will connect the `D` pin of the physical flip-flop cell directly to the power supply (`Vdd` or logic high).
---


### ► Lab 3: Toggle Flip-Flop (`dff_const3`)

This module describes a flip-flop whose data input is its own inverted output. This is the classic structure for a toggle flip-flop, which divides the clock frequency by two.

```verilog
// dff_const3.v
module dff_const3(
    input clk, 
    input reset, 
    output reg q
);
  always @(posedge clk, posedge reset) begin
    if (reset)
      q <= 1'b0;
    else
      q <= ~q; // Toggle the output on every clock edge
  end
endmodule
```
> 
<img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/1df7309b-d9ef-4f11-96a3-5182a5e417fb" />

> **Observation**: The synthesis tool is smart enough to recognize the `q <= ~q` pattern. Instead of creating a standard flip-flop and a separate inverter gate in a feedback loop, it will map this behavior directly to a specialized toggle flip-flop cell from the standard cell library. This results in a more efficient and compact implementation.
---

### ► Lab 4: Unused Logic Removal (`counter_opt`)

This final lab shows how the `opt_clean -purge` command removes logic that is not part of the final output cone. Here, a 4-bit counter is created, but only one of its bits is actually used.

```verilog
// counter_opt.v
module counter_opt(
    input clk, 
    input reset, 
    output y
);
  reg [3:0] count;

  always @(posedge clk, posedge reset) begin
    if (reset)
      count <= 4'b0;
    else
      count <= count + 1;
  end

  // Only the most significant bit is used as an output
  assign y = count[3]; 
endmodule
```
> <img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/286a4d2d-78fc-4f0c-b49d-f698d8355aa9" />

> **Observation**: The `synth` pass will initially create hardware for the full 4-bit counter (four flip-flops and associated logic). However, the `opt_clean -purge` command will trace the logic back from the output `y` and find that only the flip-flop corresponding to `count[3]` has any effect on the output. Consequently, the three unused flip-flops (`count[2:0]`) and their associated logic will be completely removed from the final netlist.
