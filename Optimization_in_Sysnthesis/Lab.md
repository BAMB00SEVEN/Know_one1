

### MODULE --- 5

# Lab: Preventing Latches and Building Scalable Logic

This lab is split into two parts. First, we will deliberately write code that infers latches and then fix it. Second, we will use loops and generate blocks to build scalable multiplexers, demultiplexers, and a Ripple-Carry Adder.

---

## Synthesis Workflow Reminder

For each lab, use the standard Yosys synthesis flow to generate and view the netlist. Pay close attention to the warnings Yosys prints—it is very good at detecting and warning you about inferred latches.

---

## Part 1: Latch Inference and Prevention (Labs 1-5 )

### ► Lab 1 & 2: Incomplete `if` Statements

This code fails to specify what `y` should be if the `if` or `else if` conditions are false.

```verilog
// Lab 1: Incomplete if
module incomp_if (input i0, i1, output reg y);
always @(*) begin
    if (i0) y = i1; // What happens if i0 is 0? -> Latch!
end
endmodule
```
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/dc7501de-f0e8-4a41-ac46-424f4eb06655" />
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/950d8333-9b92-4292-a4a0-ec877fca5ee5" />

```
// Lab 2: Incomplete nested if-else
module incomp_if2 (input i0, i1, i2, i3, output reg y);
always @(*) begin
    if (i0) y = i1;
    else if (i2) y = i3; // What happens if i0=0 and i2=0? -> Latch!
end
endmodule
````
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/f7c2a433-0160-4fe7-a1ba-9d921433f6ca" />
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/2fbc08fd-6295-46fd-b1af-a74e6491b89b" />

> **Observation**: Yosys will infer a latch for `y` and issue a warning.

---

### ► Lab 3: The Correct `case` Statement

This module correctly assigns a value to `y` in all conditions using a `default` case.

```verilog
module comp_case (input i0, i1, i2, [1:0] sel, output reg y);
always @(*) begin
    case(sel)
        2'b00 : y = i0;
        2'b01 : y = i1;
        default : y = i2; // Covers all other cases (2'b10, 2'b11)
    endcase
end
endmodule
```

><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/976ee307-31e8-42c0-80e7-82e530406c8c" />

><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/35942559-0f7a-49be-a2f4-5443bd516576" />



> **Observation**: This synthesizes cleanly to a MUX with no warnings.

### ► Lab 4 & 5: Subtle Latch Inference in `case`

Even `case` statements can infer latches if not all outputs are assigned in all branches.

```verilog
// Lab 4: Incomplete case (missing sel=2'b11)
module bad_case (input i0, i1, i2, [1:0] sel, output reg y);
always @(*) begin
    case(sel)
        2'b00: y = i0;
        2'b01: y = i1;
        2'b10: y = i2;
        // No case for 2'b11 -> Latch!
    endcase
end
endmodule

```
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/f0dc3fe6-ac85-49f9-922e-7649664f566f" />

><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/806b7cb2-61ba-4067-8a96-1ebf589b229b" />

```verilog
// Lab 5: Partial assignment in case(A bad_case)
module partial_case_assign (
    input i0, i1, i2,
    input [1:0] sel,
    output reg y,
    output reg x
);
    always @(*) begin
        case(sel)
            2'b00: begin
                y = i0;
                x = i2;
            end
            2'b01: begin
                y = i1; // 'x' is not assigned here -> Latch on x!
            end
            default: begin
                y = i2;
                x = i1;
            end
        endcase
    end
endmodule
```
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/1879e974-b7b6-4ccf-ab3d-68e505bec2d3" />
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/df9e38ff-5453-4fcb-8946-529b2e472531" />
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/eeaf8979-4053-4988-830b-87e9ab92214b" />



> **Fix**: Always include a `default` case or ensure every possible value of the `case` selector is covered. For partial assignments, assign a default value to all outputs at the beginning of the `always` block.

-----

## Part 2: Scalable Design with Loops and Generate (Labs 9-12)

### ► Lab 6 & 7: MUX/DEMUX with `for` Loop

A `for` loop is an elegant way to describe the behavior of a MUX or DEMUX.

```verilog
// Lab 6: 4-to-1 MUX with for loop
module mux_for_loop (input [3:0] i, input [1:0] sel, output reg y);
    integer k;
    always @(*) begin
        for (k = 0; k < 4; k = k + 1) begin
            if (k == sel) y = i[k];
        end
    end
endmodule
```
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/6dbed977-34e9-4dfb-8e78-d490f3a2b7b0" />
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/ec1ebe9b-8659-4e88-b3e9-92d9c838524f" />

```
// Lab 7: 8-to-1 DEMUX with for loop
module demux_for_loop (input [2:0] sel, input i, output reg [7:0] y);
    integer k;
    always @(*) begin
        y = 8'b0; // Default assignment to prevent latches
        for (k = 0; k < 8; k = k + 1) begin
            if (k == sel) y[k] = i;
        end
    end
endmodule
```
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/94080788-0bd8-40c9-9d1d-218c06813bbf" />
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/be4f6e4a-4efb-453f-9b36-9d191a5b05ea" />
------

### ► Lab 8: 8-bit Ripple-Carry Adder with `generate`

A `generate` block is perfect for creating the repetitive structure of an RCA.

```verilog
// Prerequisite Full-Adder Module
module fa (input a, b, c, output co, sum);
    assign {co, sum} = a + b + c;
endmodule

// Lab 8: 8-bit RCA
module rca (input [7:0] a, b, output [8:0] sum);
    wire [7:0] co; // Internal carry wires

    // Instantiate the first full adder (no carry in)
    fa u_fa_0 (.a(a[0]), .b(b[0]), .c(1'b0), .co(co[0]), .sum(sum[0]));

    // Generate the rest of the adder chain
    genvar i;
    generate
        for (i = 1; i < 8; i = i + 1) begin : fa_chain
            fa u_fa (.a(a[i]), .b(b[i]), .c(co[i-1]), .co(co[i]), .sum(sum[i]));
        end
    endgenerate

    assign sum[8] = co[7]; // Final carry-out
endmodule
```
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/85bda2fd-7f16-470b-87e2-f1174d30c0b9" />

><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/5a6947ec-3173-4164-9927-353c4248b059" />

> **Observation**: The `generate` block will create 7 instances of the `fa` module, named `fa_chain[1].u_fa`, `fa_chain[2].u_fa`, etc., automatically connecting the carry chain.

