# Day 5: VSDBabySoC Design and Modeling 

This document covers the design, modeling, and pre-synthesis simulation of the **VSDBabySoC**, a compact System on Chip featuring a RISC-V core.

##  Contents

- [1. Introduction to the VSDBabySoC](#1-introduction-to-the-vsdbabysoc)
  - [What is an SoC?](#what-is-an-soc)
  - [What is RVMYTH?](#what-is-rvmyth)
  - [What is a PLL?](#what-is-a-pll)
  - [What is a DAC?](#what-is-a-dac)
- [2. Project Setup and Preparation](#2-project-setup-and-preparation)
  - [Project Structure](#project-structure)
  - [Cloning the Project](#cloning-the-project)
- [3. TL-Verilog to Verilog Conversion](#3-tl-verilog-to-verilog-conversion)
- [4. Pre-Synthesis Simulation](#4-pre-synthesis-simulation)
- [5. Viewing the Waveform in GTKWave](#5-viewing-the-waveform-in-gtkwave)
  - [Viewing DAC Output in Analog Mode](#viewing-dac-output-in-analog-mode)
- [6. Key Concepts and Troubleshooting](#6-key-concepts-and-troubleshooting)
  - [Why Pre-Synthesis vs. Post-Synthesis?](#why-pre-synthesis-vs-post-synthesis)
  - [Troubleshooting Tips](#troubleshooting-tips)

---

## 1. Introduction to the VSDBabySoC

**VSDBabySoC** is a small yet powerful RISC-V-based System on Chip designed to test and calibrate three open-source IP cores together.

It integrates:
-   **RVMYTH**: A simple RISC-V-based CPU core.
-   **PLL**: An 8x Phase-Locked Loop for stable clock generation.
-   **DAC**: A 10-bit Digital-to-Analog Converter for interfacing with analog devices.

### What is an SoC?
An **SoC (System on Chip)** is a single integrated circuit that combines multiple IP (Intellectual Property) cores, such as microprocessors (digital) and modems (analog), onto a single chip.

### What is RVMYTH?
The **RVMYTH** core is a simple RISC-V-based CPU designed for educational purposes and small-scale applications, providing a practical example of a processor implementation.

### What is a PLL?
A **PLL (Phase-Locked Loop)** is a control system that generates an output signal whose phase is related to an input signal's phase. It is widely used for clock generation and synchronization.

### What is a DAC?
A **DAC (Digital-to-Analog Converter)** is a system that converts a digital signal into an analog signal, enabling communication with analog devices like speakers or screens.

---

## 2. Project Setup and Preparation

### Project Structure
The project is organized with Verilog modules, include files, and an output directory for simulation results.
-   `src/include/`: Contains header files (`.vh`).
-   `src/module/`: Contains the Verilog source files for all IPs and the testbench.
-   `output/`: Stores compiled outputs and VCD files.

### Cloning the Project
```bash
cd ~/VLSI
git clone [https://github.com/manili/VSDBabySoC.git](https://github.com/manili/VSDBabySoC.git)
cd ~/VLSI/VSDBabySoC/
```

# TL-Verilog to Verilog Conversion 
 This process is required for the `RVMYTH` core in the `VSDBabySoC` project, as it is written in TL-Verilog and must be converted before it can be used by standard simulation and synthesis tools.



---

## Conversion Steps

Follow the steps below to set up the environment and convert the `rvmyth.tlv` file.

## Step 1: Install Python Dependencies
First, ensure you have the necessary Python tools installed on your system.
```bash
sudo apt update
sudo apt install python3-venv python3-pip
```

## Step 2: Create and Activate a Virtual Environment
It's a best practice to use a virtual environment to manage project-specific dependencies.

Bash

### Navigate to the project directory
    cd ~/VLSI/VSDBabySoC/

### Create the virtual environment
    python3 -m venv sp_env

### Activate the virtual environment
    source sp_env/bin/activate
## Step 3: Install SandPiper-SaaS
With the virtual environment active, install the SandPiper tool using pip.

```bash

pip install pyyaml click sandpiper-saas
```
## Step 4: Convert TLV to Verilog
Finally, run the sandpiper-saas command to perform the conversion.

```bash

sandpiper-saas -i ./src/module/*.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir ./src/module/
```
## Verification
After running the conversion command, you can verify that the rvmyth.v file has been successfully generated in the src/module/ directory. 

```Bash

ls src/module/
```

Environment Usage Notes
To use this environment in future sessions, you must first activate it:

```Bash

source sp_env/bin/activate
```
When you are finished, you can deactivate it:

```Bash

deactivate
```
# Pre-Synthesis Simulation 

**Pre-Synthesis Simulation** is a fundamental step in the hardware verification process. Its goal is to verify the **functional correctness** of the high-level RTL code in an ideal, zero-delay environment, *before* the design is synthesized into a gate-level netlist.



---

## Key Characteristics

-   **Focus on Functionality**: It verifies that the design behaves according to the specification, without considering the timing delays of actual logic gates or wires.
-   **Zero-Delay Environment**: All logical events are assumed to happen instantaneously on the active clock edge.
-   **High Speed**: It is significantly faster than post-synthesis simulation because it does not have to model the complexity of a full gate-level netlist.

---

## Simulation Flow for VSDBabySoC

The following commands are used to run a pre-synthesis simulation for the `VSDBabySoC` project. The `-DPRE_SYNTH_SIM` flag is a macro used for conditional compilation within the testbench, and the `-I` flags specify the include directories for the Verilog compiler.


### Navigate to the project directory
    cd ~/VLSI/VSDBabySoC/

### Create the output directory if it doesn't exist
    mkdir -p output/pre_synth_sim

### Compile the design and testbench using iverilog
    iverilog -o ~/VLSI/VSDBabySoC/output/pre_synth_sim/pre_synth_sim.out \
    -DPRE_SYNTH_SIM \
    -I ~/VLSI/VSDBabySoC/src/include \
    -I ~/VLSI/VSDBabySoC/src/module \
    ~/VLSI/VSDBabySoC/src/module/testbench.v

### Navigate to the output directory and run the simulation executable
    cd output/pre_synth_sim
    ./pre_synth_sim.out

# Viewing Waveforms in GTKWave 

After running a simulation, the next step is to analyze the generated waveform to verify the design's behavior. This guide covers how to use **GTKWave** to view the Value Change Dump (`.vcd`) file from the `VSDBabySoC` simulation.


---

## Opening the VCD File

To start the waveform viewer, run the following command from the project's root directory (`~/VLSI/VSDBabySoC/`).

```bash
gtkwave output/pre_synth_sim/pre_synth_sim.vcd
```

# Key Concepts and Troubleshooting 

This document covers key verification concepts, such as the difference between pre-synthesis and post-synthesis simulation, and provides common troubleshooting tips.



---

## Why Pre-Synthesis vs. Post-Synthesis?

Understanding the difference between these two simulation stages is critical for effective hardware verification.

-   **Pre-Synthesis Simulation**:
    -   Focuses only on verifying the **functionality** of the RTL code.
    -   Operates in a **zero-delay** environment, where events occur ideally on the active clock edge.
    -   It is fast and used to catch logical bugs in the design's behavior.

-   **Post-Synthesis Simulation (GLS)**:
    -   Uses the synthesized **gate-level netlist** to simulate both **functionality and timing**.
    -   Helps identify timing violations and potential **synthesis-simulation mismatches** (e.g., unintended latches or race conditions).
    -   It is slower but provides a more accurate representation of the final hardware's behavior.

---

## Troubleshooting Tips 

-   **Module Redefinition Errors**: If you encounter errors about a module being redefined, it means the same file is being included or compiled more than once. Ensure that modules are sourced only a single time—either within the testbench or on the command line, but not both.

-   **Path Issues**: If the compiler reports that it cannot find a file, double-check that the paths specified with the `-I` flag are correct. Using full (absolute) paths instead of relative paths can often resolve these errors.



