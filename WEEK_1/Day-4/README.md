# Day 4: Gate Level Simulation (GLS) 

This section covers Gate Level Simulation (GLS), a simulation method that runs a testbench against a post-synthesis **gate-level netlist** instead of the original RTL code.

---
## Table of Contents

- [Why is GLS Important?](#why-is-gls-important)
- [GLS Flow using iverilog](#gls-flow-using-iverilog-)
- [Common Causes of Synthesis-Simulation Mismatch](#common-causes-of-synthesis-simulation-mismatch-)
  - [1. Missing Sensitivity List](#1-missing-sensitivity-list)
  - [2. Blocking vs. Non-blocking Statement Mismatch](#2-blocking-vs-non-blocking-statement-mismatch)

## Why is GLS Important?

GLS is a critical verification step for several reasons:

-   **Logical Correctness**: It verifies that the logic of the design has been correctly preserved after the synthesis process.
-   **Timing Verification**: It helps ensure that the design meets its timing requirements by simulating with gate and net delays.

---

## GLS Flow using iverilog 

Performing Gate Level Simulation involves the following steps:



1.  **Synthesize the Design**: First, the RTL code is synthesized into a gate-level netlist using a tool like Yosys.
2.  **Generate the Netlist**: Export the netlist from the synthesis tool. To create a clean, portable netlist, it's best to exclude tool-specific attributes.
    ```bash
    Example command in Yosys
    write_verilog -noattr design_netlist.v
    ```
3.  **Run the Simulation**: Use a simulator like **`iverilog`** to compile the testbench along with the generated netlist and any necessary standard cell library Verilog models.
    ```bash
    Compile the testbench, netlist, and library
    iverilog tb_design.v design_netlist.v sky130_primitives.v
    ```
4.  **Analyze Waveforms**: View the resulting waveforms in a tool like **`gtkwave`** to confirm correct functionality and timing.

A **Gate-Level Verilog model** (netlist) contains information about the design's functionality and timing in terms of interconnected standard cells.

---

## Common Causes of Synthesis-Simulation Mismatch 

Sometimes, the RTL simulation behaves differently from the GLS. This is known as a **synthesis-simulation mismatch**. The most common causes are:

-   Missing signals in a sensitivity list.
-   Incorrect use of blocking (`=`) vs. non-blocking (`<=`) assignments.
-   Use of non-standard or non-synthesizable Verilog code.

### 1. Missing Sensitivity List

In an `always` block, if a signal that should trigger a change is missing from the sensitivity list (`always @(...)`), the RTL simulator might not update the output correctly. However, the synthesis tool infers the logic based on the code's body, creating hardware that is sensitive to all inputs. GLS will expose this mismatch because the netlist will behave as the synthesized hardware does, not as the flawed RTL simulation did.

For example, in a 2x1 MUX, if an input is missing from the sensitivity list, the RTL simulation might not show an output change, but the synthesized netlist (and GLS) will.

### 2. Blocking vs. Non-blocking Statement Mismatch

The distinction between blocking (`=`) and non-blocking (`<=`) assignments is critical inside an `always` block.

-   **Blocking (`=`)**: Statements are executed sequentially, one after another. This is suitable for describing **combinational logic**. If used incorrectly to model sequential logic, it can lead to a mismatch because the simulator will wait for the first statement to finish before evaluating the next, which is not how parallel hardware works.
-   **Non-blocking (`<=`)**: Statements are scheduled to happen concurrently at the end of the time step. This is the correct way to model **sequential logic** (like flip-flops) and avoids race conditions.

Using blocking assignments where non-blocking should be used is a classic cause of synthesis-simulation mismatch, as the synthesis tool will infer different hardware than what the RTL simulation shows.

