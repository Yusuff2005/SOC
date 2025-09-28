# Day 2: Timing Libs, Hierarchical vs. Flat Synthesis, and Flop Coding Styles 

This log covers the fundamentals of standard cell libraries, synthesis methodologies, and the importance of efficient flip-flop coding styles in digital design.

## 📚 Contents

- [Liberty (.lib) Format](#liberty-lib-format)
- [Standard Cell Analysis](#standard-cell-analysis)
  - [Example: `a2111o_1`](#example-a2111o_1)
  - [Comparison: `and2_1`, `and2_2`, and `and2_4`](#comparison-and2_1-and2_2-and-and2_4)
- [Hierarchical vs. Flat Synthesis](#hierarchical-vs-flat-synthesis)


- [D Flip-Flop Coding Styles](#d-flip-flop-coding-styles)
  - [DFF with Asynchronous Reset](#dff-with-asynchronous-reset)
  - [DFF with Asynchronous Set](#dff-with-asynchronous-set)
  - [DFF with Synchronous Reset](#dff-with-synchronous-reset)
  - [DFF with Both Asynchronous and Synchronous Reset](#dff-with-both-asynchronous-and-synchronous-reset)
- [Synthesis of DFF Styles](#synthesis-of-dff-styles)
- [Interesting Synthesis Optimisations](#interesting-synthesis-optimisations)
  - [Synthesis of `mul2`](#synthesis-of-mul2)
  - [Synthesis of `mult8`](#synthesis-of-mult8)

---

## Liberty (.lib) Format

The **Liberty (`.lib`) format** is an industry standard for describing the characteristics of library cells for a specific technology. A library file contains critical information for synthesis, placement, and timing analysis tools.

The information inside a `.lib` file can be divided into two main parts:
- **Common Information**: This first part contains global information applicable to all cells in the library, such as:
  - Library and technology name.
  - Units for time, power, voltage, etc.
  - Operating conditions (process, voltage, temperature) for different corners (e.g., Max, Min, Typical).
- **Cell-Specific Information**: The second part details the properties of each individual standard cell, including timing, power, area, and functionality.

---

## Standard Cell Analysis

### Example: `a2111o_1`
The cell `a2111o_1` from the `sky130_fd_sc_hd__tt_025C_1v80.lib` is a complex gate that implements the following logic function: `X = !((A1 & A2) | B1 | C1 | D1)`.

### Comparison: `and2_1`, `and2_2`, and `and2_4`
The Sky130 library provides multiple "flavors" of the same standard cell. For example, `and2_1`, `and2_2`, and `and2_4` are all 2-input AND gates, but they have different drive strengths, allowing the synthesis tool to choose the most appropriate one based on the load it needs to drive.

---

## Hierarchical vs. Flat Synthesis

**Hierarchical design** is a "divide and conquer" approach where a large design is broken down into smaller, manageable sub-modules. This helps tools optimize smaller blocks more efficiently, improves resource utilization, and speeds up runtime and debugging.

**Flat synthesis** collapses the entire design hierarchy into a single top-level module. This approach is suitable for smaller designs where the tool can handle the entire logic block efficiently.

#### Design: `multiple_modules.v`
```verilog
module sub_module1(i1, o1);
  input i1;
  output o1;
  assign o1 = i1;
endmodule

module sub_module2(i2, o2);
  input i2;
  output o2;
  assign o2 = i2;
endmodule

module multiple_modules(a, b, c, d);
  input a, b;
  output c, d;
  sub_module1 sm1(a, c);
  sub_module2 sm2(b, d);
endmodule
```
## D Flip-Flop Coding Styles

### DFF with Asynchronous Reset
In this style, the reset signal has higher priority and acts immediately, regardless of the clock. If the reset is active, the output is forced to its reset state. The clock edge only influences the output when the reset is inactive.

### DFF with Asynchronous Set
Similar to an asynchronous reset, an asynchronous set acts immediately and independently of the clock. When the set signal is active, the output is forced to a '1'.

### DFF with Synchronous Reset
A synchronous reset is only evaluated at the active clock edge. If the reset signal is active *at the same time* as the clock edge, the flip-flop is reset. At all other times, the reset signal is ignored.

### DFF with Both Asynchronous and Synchronous Reset
This design combines both behaviors. The asynchronous reset has the highest priority and can reset the flop at any time. The synchronous reset is only considered at the clock edge, and only if the asynchronous reset is inactive.

---

## Synthesis of DFF Styles

The synthesis flow for flip-flops uses a special command, **`dfflibmap`**. This step is crucial for mapping the generic DFF cells inferred by Yosys to the specific, technology-dependent flip-flop cells available in the provided standard cell library.

```bash
# Generic Yosys flow for DFF synthesis
yosys
read_liberty -lib <path_to_liberty_file>
read_verilog <dff_design.v>
synth -top <module_name>
dfflibmap -liberty <path_to_liberty_file>
abc -liberty <path_to_liberty_file>
show
```
# Interesting Synthesis Optimisations 

## Understanding Synthesis Optimisation 

**Synthesis optimisation** is the process where a tool like Yosys analyzes the RTL code and simplifies it to produce the most efficient hardware possible. This involves removing redundant logic, simplifying complex mathematical expressions, and reducing the final gate count. The goal is to improve the design's area, speed, and power consumption without changing its core functionality.

A common example is how tools handle multiplication by a constant that is a power of two.

---

## Example 1: Synthesis of `mul2`

When the synthesis tool encounters code that multiplies a variable by 2 (e.g., `y = a * 2`), it does not implement a complex and resource-intensive multiplier circuit.

Instead, it recognizes that multiplying by 2 is mathematically equivalent to a **left bit-shift by 1 bit**. This operation is extremely simple in hardware and can be implemented using just wires, requiring **no standard cell logic gates** at all.

#### Synthesis Flow in Yosys
```bash
yosys
read_liberty -lib <path_to_liberty_file>
read_verilog mult_2.v
synth -top mul2
show
write_verilog -noattr mult2_netlist.v
```
# Synthesis of a Multiplier by 8 (`mult8`) 


## Verilog RTL Code

The Verilog code for the `mult8` module is straightforward. It takes an 8-bit input `a` and assigns the output `y` to the value of `a` multiplied by 9. But sysnthesize gives `aa`.

```verilog
module mult8(
    input [7:0] a,
    output [7:0] y
);

    assign y = a * 9;

endmodule
```


