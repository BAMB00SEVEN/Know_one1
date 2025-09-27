# Day 2 Theory: Libraries, Synthesis, and Resets

This guide covers the core concepts for turning RTL into a gate-level circuit: the cell datasheets (`.lib` files), the high-level build strategies (synthesis), and the design of state-holding elements (flip-flops).

---

## 1. Timing Libraries (`.lib`)

A **`.lib` file** is a digital datasheet for every standard cell (like AND, OR, FF) in a technology. The synthesis tool requires this file to know the timing, power, and area characteristics of the building blocks it can use.

The filename `sky130_fd_sc_hd__tt_025C_1v80.lib` specifies a **PVT Corner**:
* **Process (`tt`)**: Assumes a **Typical** manufacturing process.
* **Voltage (`1v80`)**: Assumes a **1.80V** power supply.
* **Temperature (`025C`)**: Assumes an operating temperature of **25°C**.

Designs must be validated across various PVT corners to ensure they work reliably in the real world.

---

## 2. Synthesis Strategies

Synthesis converts RTL into a gate-level netlist. The two primary strategies offer a trade-off between optimization and simplicity.

| Aspect | Hierarchical Synthesis | Flattened Synthesis |
| :--- | :--- | :--- |
| **Analogy** | Building with pre-assembled modules. | Building from one giant pile of parts. |
| **Structure** | Preserves your `module` boundaries. | Dissolves all modules into one. |
| **Advantage** | Faster for large designs; easy to debug. | Achieves maximum global optimization. |
| **Disadvantage** | Limited cross-module optimization. | Slow for large designs; hard to debug. |

---

## 3. Flip-Flop Reset Types

The key difference is how a flip-flop responds to a reset signal: instantly or in sync with the clock. This is controlled by the `always` block's **sensitivity list**.

### ► Asynchronous Reset (The Override)
This is an "emergency stop" that acts immediately, ignoring the clock.

* **Mechanism**: The reset signal is in the sensitivity list: `always @ (posedge clk, posedge async_reset)`.
* **Behavior**: The output is forced to its reset state the moment `async_reset` becomes active.
* **Use Case**: Best for system-wide power-on resets.

### ► Synchronous Reset (The Orderly Command)
This reset is a planned event that waits for the next clock tick.

* **Mechanism**: Only the clock is in the sensitivity list: `always @ (posedge clk)`.
* **Behavior**: The tool checks the reset signal's value only on the active clock edge.
* **Use Case**: Preferred for most internal logic to ensure predictable, synchronous behavior.
* **Behavior**: The flip-flop only checks the value of the reset signal on the rising edge of the clock. The reset action is synchronized with the rest of the circuit's activity.
* **Use Case**: Generally preferred for resets within a clocked logic block because it prevents timing issues and ensures predictable, synchronous behavior.
