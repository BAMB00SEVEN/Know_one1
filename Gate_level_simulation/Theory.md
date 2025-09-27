

### Verification, Mismatches, and Verilog's Two Assignments

Today's theory focuses on what happens *after* synthesis. We'll explore how to verify the synthesized circuit, why it might not match your original design, and the single most common reason for such errors: the misuse of Verilog's assignment operators.

---

## 1. Gate-Level Simulation (GLS)

**The Idea**: Verifying the final, gate-level netlist produced by the synthesis tool, not just your original RTL code.

Think of it this way:
* **RTL Simulation** is like checking the architect's **blueprint** for a house.
* **Gate-Level Simulation** is like inspecting the **actual, physical house** after the construction crew has built it.

GLS is a critical check to ensure the "construction crew" (the synthesis tool) correctly interpreted your blueprint and that the final product works under real-world conditions.

### ► Key Questions GLS Answers

1.  **Functional Correctness**: Does the gate-level circuit do the same thing as my RTL?
2.  **Timing Verification**: Will the circuit work at the target clock speed? GLS uses real gate and wire delay information to check for timing violations (setup/hold issues).
3.  **Testability Check**: Did features added for manufacturing tests, like scan chains, get implemented correctly?

---

## 2. Blocking (`=`) vs. Non-Blocking (`<=`) Assignments

This is the most critical concept for writing correct, synthesizable Verilog. The operator you choose tells the simulator and synthesizer what kind of hardware to build.

### ► The Golden Rules
1.  Use **Blocking (`=`)** for **Combinational Logic** (in `always @(*)` blocks).
2.  Use **Non-Blocking (`<=`)** for **Sequential Logic** (in `always @(posedge clk)` blocks).

### ► An Analogy: Following a Recipe

* **Blocking (`=`) is Sequential**: Imagine a recipe where you must complete each step in order. You compute Step 1, get a result, and then use *that new result* for Step 2. The statements execute one after another, immediately.
    ```verilog
    // In a blocking block, 'd' uses the NEW value of 'x'
    x = a | b; // Step 1: Happens immediately
    d = x & c; // Step 2: Happens immediately after
    ```

* **Non-Blocking (`<=`) is Concurrent**: Imagine a team of chefs. At the "tick" of a clock, they all read the *original* ingredients, go prepare their part of the meal, and then update the final dish *all at the same time* at the end of the tick. All assignments are scheduled to happen together at the end of the current time step.
    ```verilog
    // In a sequential block, Q1 and Q2 get their new values simultaneously
    // at the clock edge, using the old values of the inputs.
    Q1 <= D1;
    Q2 <= D2;
    ```

Mixing these up is the primary cause of simulation-synthesis mismatches.

---

## 3. Synthesis-Simulation Mismatch

**The Idea**: When your RTL simulation passes, but the gate-level simulation (or the final chip) fails. The pre-synthesis and post-synthesis behaviors do not match.

This happens because the simulator and the synthesis tool **interpreted your code differently**.

### ► Common Causes

* **Wrong Assignment Operator**: Using blocking (`=`) in sequential logic is a classic cause of race conditions that simulate differently than they synthesize.
* **Incomplete Sensitivity Lists**: Forgetting a signal in a combinational `always` block sensitivity list can make it simulate like a latch, while synthesis may behave differently. Using `always @(*)` solves this.
* **Non-Synthesizable Code**: Using Verilog constructs that have no hardware equivalent, such as `#10` delays or `$display` statements. Simulators understand these, but synthesis tools ignore them.

