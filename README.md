# Summary
---
I got a basic idea about flow of chip design. I am going to give a summary about what I understood.

⦁	At first he explained about GCC which is the compiler to run C in Linux

⦁	Then spec is the one which is the list of  requirements, the chip should meet. So the second step is to check the specifications whether it can be implementable or not. So it can be check by C.

---

To implement the hardware via software. For that we use verilog which is the hardware descriptive language to test the specifications are to be meet in hardware.

This step is divided into two:
1) Processor
2) Peripherals
---
1) In processor

⦁	Gate level design wil be executed.

⦁	In this design, The verilog code needs to be synthesizable. It should not contain any constructs.

---

2) In Peripherals

⦁	Macros and Analog IP's have to execute. Macros is the reusable and synthesizable block.

⦁	Whereas Analog IP's are Non synthesizable block.

⦁	Then the SOC Engineer rechecks the result

⦁	Then it will send to fabrication foundry called tapeout. After manufacturing process, foundry returns the physical chip called tape in.

⦁	After the chip is manufactured it fits in the device which fits for its clock frequency and checks the output.

---
⦁	In every step, all the output should result the same. Otherwise, the distinct result process should have to work again from the beginning.
