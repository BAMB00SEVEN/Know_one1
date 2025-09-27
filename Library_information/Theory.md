###  MODULE --- 2
------

# Theory: Libraries, Synthesis, and Resets

This guide covers the core concepts for turning RTL into a gate-level circuit: the cell datasheets (`.lib` files), the high-level build strategies (synthesis), and the design of state-holding elements (flip-flops).

---

## 1. Timing Libraries (`.lib`)

A **`.lib` file** is a digital datasheet for every standard cell (like AND, OR, FF) in a technology. The synthesis tool requires this file to know the timing, power, and area characteristics of the building blocks it can use.

The filename `sky130_fd_sc_hd__tt_025C_1v80.lib` specifies a **PVT Corner**:
* **Process (`tt`)**: Assumes a **Typical** manufacturing process.
* **Voltage (`1v80`)**: Assumes a **1.80V** power supply.
* **Temperature (`025C`)**: Assumes an operating temperature of **25°C**.

><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/422ee6b6-0b26-4140-9642-fa55037ca2c8" />

Designs must be validated across various PVT corners to ensure they work reliably in the real world.

**for finding your desired cell press `shift + :` , you will reach the command line of the editor andnthen type `/cell .*your_cell_name` , also you can use `vsp` command to split screen**

><img width="2548" height="1458" alt="Image" src="https://github.com/user-attachments/assets/e4753faf-e2b1-4b81-97ff-936d87a22a13" />

**we can also see that for as the number of outputs is increasing in the selected nand gate the cells are getting wider and wider cells consume more area and power**

><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/604f0789-c9e9-4e1a-837b-6a6521e8b03d" />
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

`The netlist for heirarchal synthesis looks like`

><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/393904b6-5823-46bd-8286-2d2f86abfb41" />
*here the heirarchy is maintained as there is a assignment of seperate modules*

`The netlist of flattend synthesis looks like`

><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/de99fc41-2125-49c9-abfe-9bc888390879" />
*here we can see that everything came down to gate level*

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
