---
layout: post
title:  "Major SoCs are heterogeneous-ISA machines"
date:   2025-01-25 21:50:52 -0500
category: blog
tags: chip design microprocessor ISA 
---

A typical System-on-Chip (or SoC) in a modern smartphone, tablet, or a laptop will have a structure like this:  
  

![SoC Diagram](https://i.imgur.com/2FXgNkk.png){:width="85%" style="display:block; margin-left:auto; margin-right:auto"}

  
You may have one ore more CPU cores, some number of GPU cores, a multimedia controller (for hardware accelerated video encode/decode). Modern SoCs may also include a neural network engine (NPU) for inferencing. The chip will also include one or more memory controllers (for DRAM) and several I/O controllers (for WiFi, Bluetooth, cellular modem, etc.). Typically, there will also be controllers for various chip management functions such as security, thermal management, power management, etc.

You may think your device is based on an Instruction Set Architecture (ISA) such as x86/AMD64, or ARM, or RISC-V, etc., and that it is the only ISA that is being used. But, in reality, the machine is likely running multiple ISAs. Or, at the very least, multiple variants of an ISA.

Let's examine this in greater detail.
	
### ISAs ###

Broadly speaking, the [ISA](https://en.wikipedia.org/wiki/Instruction_set_architecture "Instruction Set Architecture") defines characteristics of the processor instructions such as instruction types, opcode encoding, and registers. It also defines memory related features such as addressing modes, memory consistency, and (where supported) virtual memory. Instructions can be defined as user accessible or reserved for privileged software (operating system, hypervisor, etc.). 

The different engines within the SoC can all be running their own ISAs. 

*CPU:*
* The main ISA of the CPU could be x86, ARM, RISC-V, etc.
* If the processor has different kind of cores (say ARM's big.LITTLE or Performance-cores and Efficiency-cores in other designs), then there might be a difference in the ISA supported by those core variants

*GPU:*
* GPUs execute code built using vendor-specific Single-Instruction Multiple-Data (SIMD) ISAs
* There might be specialized cores within the GPU (e.g. tensor cores) that might have a different set of instructions compared to actual graphics processing cores

*Multimedia Engine:*
* Various Digital Signal Processing (DSP) cores are used for audio processing, which use their own set of instructions
* Common video encode/decode engines also employ a small microprocessor running firmware along with fixed function blocks for operations like Fourier and Discrete Cosine transforms

*Neural Engine:*
* Several modern processors include an NPU which typically have a set of custom cores for inference
* These engine cores have their own ISA along with local caches and scratch memory

*System Management:*
* A lot of the larger SoCs will use microcontrollers from ARM, Tensilica, AVR, PIC, etc. for a lot of their system management functions
* This includes power management, thermal management etc.

*Security processor:*
* Modern SoCs also have a separate security processor that is utilized for root of trust, secure boot, and for providing a separate trusted execution environment (TEE). Examples are ARM's TrustZone and AMD Platform Security Processor (PSP).

*I/O Controllers:*
* WiFi, Bluetooth controllers generally also have microcontrollers in their IP blocks

Companies might try to limit the number of different ISAs they use in order to streamline their firmware development flow, and to simplify their IP licensing process (most of these other controllers will use 3rd party cores that are licensed).

#### Code ####

We have now seen how the SoC can have several engines, each running their own ISA. But how do these different engines get the code they are supposed to execute? Where does this code reside?

*CPU:*
The code running on the main CPU would be spread between the boot code (BIOS, UEFI, etc.) or in the operating system, hypervisor, drivers and ultimately the end-user applications.

*GPU:*
The code for the GPU is in the graphics application and some of it could be generated on the fly by the JIT compiler typically included along with the graphics driver.

*Multimedia, Neural Engine:*
Code is shipped along with the device driver, and the driver loads the code into the engine's RAM

*System Management:*
The firmware coode for the system management functions is usually bundled with system boot code, and gets loaded into an enclave in system memory or loaded into SRAM built next to the microcontroller.

*Security processor:*
The code for the security processor is cryptographically signed and shipped along with the platform firmware in the boot flash. Some of the code may also be etched in ROM. The ROM code would get executed first, which then loads and verifies the next stage of the firmware from boot flash.

Other firmware may exist in the BIOS or the OS/drivers.


### Conclusion ###

Every major SoC is a heterogeneous ISA multiprocessor machine, running code with different instructions based on the tasks and the hardware accelerators in use. Each of these ISAs may have different memory consistency model semantics, and it is the job of the SoC interconnect to ensure that all the ordering requirements are met. Each of the different microcontrollers in use may have their own caches, but for simplicity, there is generally no hardware supported cache coherency between these caches -- it is typically left up to the software to manage the coherency if there is any direct sharing of data between them.

How many different compilers do you think are used to build all this? Or do you think it's GCC for everything?
