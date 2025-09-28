# Day 3: Combinational and Sequential Optimizations 

This log covers logic optimization techniques used by synthesis tools to simplify designs, focusing on improving area and power for both combinational and sequential logic.

##  Contents

- [Combinational Logic Optimization](#combinational-logic-optimization)
  - [Techniques](#techniques)
  - [Optimization in Yosys: `opt_clean`](#optimization-in-yosys-opt_clean)
- [Sequential Logic Optimization](#sequential-logic-optimization)
- [Combinational Optimization Labs](#combinational-optimization-labs)
  - [Lab 1-4: Basic Optimizations](#lab-1-4-basic-optimizations)
  - [Lab 5: Hierarchical Optimization Labs](#lab-5--6-optimizing-hierarchical-designs)
  
- [Sequential Optimization Labs](#sequential-optimization-labs)
  - [Lab 1-5: Sequential Constant Propagation](#lab-1-5-sequential-constant-propagation)
  - [Lab 6: Optimizing Unused Outputs](#lab-6-optimizing-unused-outputs)
  - [Lab 7: No Optimization (All Outputs Used)](#lab-7-no-optimization-all-outputs-used)

---

## Combinational Logic Optimization

**Objective**: To squeeze and simplify logic to get the most optimized design in terms of **area** and **power**.

### Techniques

-   **Constant Propagation**: Replaces logic blocks with constant values when inputs are fixed. For example, if `Y = !((A & B) | C)` and `A` is tied to `0`, the expression simplifies to `Y = !C`, reducing a complex gate to a single inverter.
-   **Boolean Logic Optimization**: Simplifies complex Boolean expressions using methods like Karnaugh Maps (K-Maps) or the Quine-McCluskey algorithm. For example, `assign y = a ? (b ? c : (c ? a : 0)) : !c;` can be optimized to `y = a ^ c;`.

### Optimization in Yosys: `opt_clean`
The primary command to perform logic optimization in Yosys is **`opt_clean`**.

-   This command identifies and removes unused wires and cells.
-   Using the `-purge` switch also removes internal nets that have a public name.
-   For hierarchical designs, the design must first be flattened using the **`flatten`** command before running `opt_clean` for global optimizations.

---

## Sequential Logic Optimization

-   **Basic - Sequential Constant Propagation**: Propagates known constant values through flip-flops, replacing a flop with a constant driver (`GND` or `VCC`) if its input is fixed.
-   **Advanced Techniques**:
    -   **State Optimization**: Reduces the number of states in Finite State Machines (FSMs).
    -   **Retiming**: Moves registers across combinational logic to balance delay paths and improve timing.
    -   **Sequential Logic Cloning**: Duplicates registers to resolve high-fanout nets, improving timing and reducing congestion.

---

## Combinational Optimization Labs

### Lab 1-4: Basic Optimizations
These labs demonstrate the use of `opt_clean` on various single-module designs.

```bash
# Generic flow for labs 1-4
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog <verilog_file.v> 
synth -top <top_module>
opt_clean -purge # Removes unused or redundant logic
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```
## Hierarchical Optimization Labs

### Lab 5 & 6: Optimizing Hierarchical Designs
These labs demonstrate the proper two-phase flow required to optimize a hierarchical design with multiple sub-modules. For global optimizations to be effective, the design must first be flattened.

The process is:
1.  **Phase 1**: Synthesize the hierarchical design and then use the `flatten` command to collapse all sub-modules into a single top-level module. Write this flattened netlist to a new file.
2.  **Phase 2**: Start a new Yosys session, read the flattened netlist, and then run the `opt_clean` command to perform optimizations on the entire design.

```bash
# Phase 1: Flatten the design
yosys
read_liberty -lib <path_to_liberty_file>
read_verilog <hierarchical_design.v>
synth -top <top_module>
flatten # Essential for multi-module optimization
write_verilog -noattr <design_flat.v>

# Phase 2: Optimize the flattened netlist
yosys
read_liberty -lib <path_to_liberty_file>
read_verilog <design_flat.v>
synth -top <top_module>
opt_clean -purge # Cleans up redundant logic
abc -liberty <path_to_liberty_file>
show
```
# Sequential Optimization Labs 



---

### Lab 1-5: Sequential Constant Propagation

These labs explore how Yosys handles constant values that are fed into flip-flops during synthesis. The tool can often simplify the logic significantly.

-   **DFF is Inferred (Labs 1, 3, 5)**: A flip-flop is implemented in the final netlist because its output `q` is dependent on a changing internal state or the output of another flip-flop. The sequential logic is preserved to maintain the correct behavior.
-   **DFF is Optimized Away (Labs 2, 4)**: No flip-flop is implemented. The synthesis tool determines that the output is always a constant value (e.g., `1'b1`), regardless of the clock or reset inputs. The flop is therefore removed and replaced by a direct connection to a power line (`VCC`) or ground (`GND`).

#### Generic Synthesis Flow
```bash
yosys
read_liberty -lib <path_to_liberty_file>
read_verilog <dff_const_file.v>
synth -top <top_module>
dfflibmap -liberty <path_to_liberty_file>
abc -liberty <path_to_liberty_file>
show
```

## Lab 6: Optimizing Unused Outputs

This lab demonstrates how the **`opt_clean`** command removes unnecessary sequential logic. The RTL describes a 3-bit up-counter, but only the least significant bit (`count[0]`) is actually connected to an output.

Since `count[0]` simply toggles on every clock cycle, the `opt_clean` command recognizes that the two more significant bits (`count[1]` and `count[2]`) and their associated increment logic are functionally unused. It optimizes the entire 3-bit counter down to a **single, efficient toggle flip-flop**.

### Synthesis Flow
```bash
yosys
read_liberty -lib <path_to_liberty_file>
read_verilog counter_opt.v
synth -top counter_opt
dfflibmap -liberty <path_to_liberty_file>
opt_clean -purge # This is the key optimization step
abc -liberty <path_to_liberty_file>
show
```
# Lab 7: No Optimization (All Outputs Used) 

## Design Concept

The design consists of a 3-bit up-counter. Unlike previous labs where some counter bits were unused, here **all three bits** (`count[2:0]`) are required to generate the final output signal, `q`. The logic is designed such that `q` becomes active only when the counter reaches a specific value (e.g., when `count == 3'b100`).

---

## Expected Synthesis Result

Because all three flip-flops of the counter are needed to correctly determine the state and produce the output `q`, **no optimization can be performed**. The synthesis tool correctly preserves all the logic and implements all three flip-flops as described in the RTL.

This lab contrasts with scenarios where unused logic can be safely identified and removed by the `opt_clean` command.

---

## Synthesis Flow in Yosys

The standard synthesis flow is used to convert the RTL to a gate-level netlist. The `show` command will visually confirm that all three flip-flops and the associated combinational logic have been implemented.

```bash
yosys
read_liberty -lib <path_to_liberty_file>
read_verilog counter_opt2.v
synth -top counter_opt2
dfflibmap -liberty <path_to_liberty_file>
abc -liberty <path_to_liberty_file>
show
```
