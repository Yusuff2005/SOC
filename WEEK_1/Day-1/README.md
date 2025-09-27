# DAY - 1 : Introduction to Digital Design and Synthesis 

This document covers the fundamental concepts of digital design, including the roles of simulators, testbenches, and synthesizers in the hardware development workflow.

## Table of Contents

- [Core Concepts in Digital Verification](#core-concepts-in-digital-verification)
  - [Simulator](#simulator)
  - [Design](#design)
  - [Testbench](#testbench)
  - [How a Simulator Works](#how-a-simulator-works)
- [Introduction to Synthesis](#introduction-to-synthesis)
  - [Synthesizer](#synthesizer)
  - [RTL vs. Netlist](#rtl-vs-netlist)
- [Simulation and Synthesis Flow](#simulation-and-synthesis-flow)

---

## Core Concepts in Digital Verification

### Simulator 
A **simulator** is a tool used to check and validate the functionality of a hardware design.

### Design
The **design** consists of a set of Verilog code that describes the intended functionality of the hardware.

### Testbench
A **testbench** is used to test the design by applying different input values (stimuli) according to the design's specification.

### How a Simulator Works
The simulator's primary function is to monitor the inputs of a design. It only evaluates and updates the outputs when it detects a change in the input values.

-   A **simulator** has primary inputs and outputs that correspond to the design's ports.
-   A **testbench**, on the other hand, does not have primary inputs or outputs as its role is to generate stimulus internally.
![image_alt](https://github.com/Yusuff2005/SOC/blob/main/WEEK_1/Day-1/Images/139537557-826c48eb-a0c9-412a-8ee2-7e889b9fa562.png?raw=true)

---

## Introduction to Synthesis 

### Synthesizer
A **synthesizer** is a tool that converts the high-level Register Transfer Level (RTL) code into a low-level, gate-level netlist.

### RTL vs. Netlist
-   **RTL (Register Transfer Level)**: A high-level, abstract representation of the hardware design, focusing on behavior.
-   **Netlist**: A low-level, structural representation of the design, describing the specific logic gates and their interconnections.
![image_alt](https://github.com/Yusuff2005/SOC/blob/main/WEEK_1/Day-1/Images/VirtualBox_Ubuntu_27_09_2025_16_57_13.png?raw=true)

---

## Simulation and Synthesis Flow

Here are the basic steps to verify and synthesize a Verilog design.

1.  **Run the simulation** using `iverilog`. This command compiles your design and testbench files.
    ```bash
    iverilog your_design.v your_testbench.v 
    ```
    ![image_alt](https://github.com/Yusuff2005/SOC/blob/main/WEEK_1/Day-1/Images/VirtualBox_Ubuntu_27_09_2025_16_59_56.png?raw=true)
2.  **Generate the VCD file**. Running the compiled output (by default, `a.out`) produces a Value Change Dump (`.vcd`) file, which contains the waveform data.
    ```bash
    ./a.out
    ```

3.  **View the waves** using `gtkwave`. Open the generated VCD file to visually inspect the signals and verify the design's behavior.
    ```bash
    gtkwave your_waveform.vcd
    ```
    ![image_alt](https://github.com/Yusuff2005/SOC/blob/main/WEEK_1/Day-1/Images/VirtualBox_Ubuntu_27_09_2025_16_59_42.png?raw=true)
4.  **Synthesize the design** using a tool like `yosys`. This step converts your verified RTL code into a gate-level netlist.
    ```bash
    yosys
    ```
  ![image_alt](https://github.com/Yusuff2005/SOC/blob/main/WEEK_1/Day-1/Images/VirtualBox_Ubuntu_27_09_2025_17_00_20.png?raw=true)
    ```bash
    read_liberty -lib location
    ```
    ```bash
    read_verilog design.v
    ```
    ```bash
    synth -top module_name
    ```
  ![image_alt](https://github.com/Yusuff2005/SOC/blob/main/WEEK_1/Day-1/Images/VirtualBox_Ubuntu_27_09_2025_17_01_04.png?raw=true)
    ```bash
    abc -liberty location
    ```
    ```bash
    show
    ```
  ![image_alt](https://github.com/Yusuff2005/SOC/blob/main/WEEK_1/Day-1/Images/VirtualBox_Ubuntu_27_09_2025_17_01_33.png?raw=true)

  ![image_alt](https://github.com/Yusuff2005/SOC/blob/main/WEEK_1/Day-1/Images/VirtualBox_Ubuntu_27_09_2025_17_01_51.png?raw=true)

  ![image_alt](https://github.com/Yusuff2005/SOC/blob/main/WEEK_1/Day-1/Images/VirtualBox_Ubuntu_27_09_2025_17_02_09.png?raw=true)

  


